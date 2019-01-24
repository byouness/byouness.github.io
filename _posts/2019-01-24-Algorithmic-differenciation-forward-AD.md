---
layout: post
title: Algorithmic Differentiation - 1. Forward AD
feature-img: "assets/img/pexels/cpu.jpg"
tags: [Python, AD, AAD, Algorithmic differentiation, Operator overloading, Dual numbers]
---

This blog post attempts to give an introduction to algorithmic differentiation. It tackles forward AD, while backward AD will be the subject of another one.

## What is AD, and what is it for?

Let's consider a computer program that takes some inputs $$X$$, makes some computations and returns some outputs $$Y$$.

Mathematically speaking, this program is a function $$F$$ that is a composition of many functions representing the computation steps:

$$
	Y = F(X) = \left( f_p \circ \cdots \circ f_1 \circ f_0 \right) (X) = f_p \left( \cdots \left(f_1 \left( f_0( X ) \right) \right) \right)
$$

Now, let's suppose we need the sensitivities, or derivatives, of this function with respect to the inputs. This is very often the case in many real life applications: in optimisation routines, for sensitivity analysis, in finance for hedging and regulatory computations, etc. 

Algorithmic differentiation is a clever way to enrich our initial program $$F$$, to obtain an enhanced one that computes not only the value $$F(X)$$, but also its sensitivities $$F^\prime(X)$$.

It is used by all machine learning libraries such as Tensorflow, and PyTorch and is getting a lot of traction in finance due to the increasing complexity of the computations needed.

Unlike numeric differentiation that computes an approximation of the derivative ($$f^\prime(x) \approx \frac{f(x+h) - f(x)}{h}$$, or even better: $$\frac{f(x+h) - f(x-h)}{2h}$$ for $$h$$ sufficiently small), algorithmic differentiation computes the exact derivative (up to floating point precision of course).

In the general case, both the input and the output of the function are multidimensional.

$$
F: X = \left( \begin{array}{c}  x_1 \\ \vdots \\ x_n \end{array} \right) \mapsto \left( \begin{array}{c} F_1(x_1,\cdots,x_n) \\ \vdots \\ F_m(x_1, \cdots , x_n) \end{array} \right)
$$

In this case the derivative is actually a matrix, called the Jacobian:

$$
F^\prime(X) = \left( \begin{array}{ccc} 

\frac{\partial F_1}{\partial x_1}(X) & \cdots & \frac{\partial F_1}{\partial x_n}(X) \\ \vdots & \ddots & \vdots \\ \frac{\partial F_m}{\partial x_1}(X) & \cdots & \frac{\partial F_m}{\partial x_n}(X)

\end{array} \right)
$$

For its simplicity, we will keep the $$F^\prime(X)$$ notation nevertheless.

A column of the Jacobian gives the derivatives of all outputs w.r.t. a single input. Similarly, a row of the Jacobian gives the derivatives of a single output of the function w.r.t. each input, it is called a gradient.

Algorithmic differentiation comes in two flavors:

|--------------|---------------------------------|--------------|
| AD type 	   | Computes in each run            | For our $$F$$  |
|--------------|---------------------------------|--------------|
| __Forward__                 ( __AD__ )  | a column of the Jacobian matrix | $$ (\frac{dF_k}{dx_j})_{1 \leq k \leq m}$$ for a fixed $$j$$ |
| __Backward__ or __Adjoint__ ( __AAD__ ) | a row of the Jacobian matrix    | $$ (\frac{dF_k}{dx_j})_{1 \leq j \leq n}$$ for a fixed $$k$$ |

Before explaining how each one works and how it could be implemented, let's go through some mathematics to help us grasp the theory behind, starting with the foundation of it all: the chain rule.

## The chain rule

The chain rule gives the derivative of a composition of two functions:

$$
	(f \circ g)^\prime(x) = g^\prime(x) \times (f^\prime \circ g)(x)
$$

Applied to our function $$F$$ which consists in $$p$$ compositions, it writes:

$$
	F^\prime(X) = f^\prime_p(X_{p-1}) \times f^\prime_{p-1}(X_{p-2}) \times \cdots \times f^\prime_1(X_0)
$$

Where:

- $$X_0 = X$$;
- $$X_1 = f_1(X_0)$$;
- $$X_2 = f_2(X_1) = (f_2 \circ f_1)(X_0))$$;
- ...
- $$X_{p-1} = f_{p-1}(X_{p-2}) = (f_{p-1} \circ f_{p-2} \cdots \circ f_1)(X_0)$$. 

This is the foundation on which the AD relies.

## Relevant variables and dependencies

In practice, $$F$$ depends on a multitude of variables, among which only a small subset is realy important. This needs to be taken into account to simplify the AD or AAD. It is crucial to identify which variables (our $$X_k$$ above ) are relevant and have to be taken into account.

Indeed, some intermediary variables might not depend on our input $$X$$, and some might have no influence on the end result (in terms of derivatives).

We thus need to consider only intermediary variables which:

- depend on the initial variables of interest;
- and have an influence on the end result.

In the following, we will suppose that this is already done and that all $$(X_k)_{1 \leq k \leq p}$$ are relavant.

## Dual numbers

Theoretically speaking, algorithmic differentiation is achieved by replacing the real numbers by so-called [dual numbers](https://en.wikipedia.org/wiki/Dual_number). In this case, they can be viewed as pairs constituted by the value and derivative $$\langle f, f^\prime \rangle$$.

Computer programs are based on a limited set of operations, and functions ($$+, -, \times, /, \exp, \log, \sin, \cos, ...$$).

These basic mathematic functions and operations are redefined for this new algebra. For the first element of the pair, nothing new, it's the usual arithmetic of real numbers is used. For the second element, derivatives relations are exploited (such as the derivative of a sum, product, quotient, etc.) to get the following arithmetic:

|----------------------|--------------------------------|
| Operator or function | Relation for dual numbers      |
|----------------------|--------------------------------|
| $$+$$      | $$\langle f, f^\prime \rangle + \langle g, g^\prime \rangle = \langle f + g, f^\prime + g^\prime \rangle$$ |
| $$-$$      | $$\langle f, f^\prime \rangle - \langle g, g^\prime \rangle = \langle f - g, f^\prime - g^\prime \rangle$$ |
| $$\times$$ | $$\langle f,f^\prime \rangle \times \langle g,g^\prime\rangle = \langle f g, f^\prime g + f g^\prime \rangle$$ |
| $$/$$      | $$\frac{\langle f,f^\prime \rangle}{\langle g,g^\prime \rangle} = \left \langle \frac{f}{g}, \frac{f^\prime g - fg^\prime}{g^2}  \right\rangle \quad (g \ne 0)$$ |
| constant $$c$$  | $$\langle c, 0 \rangle$$ 
| $$\sin$$   | $$\sin(\langle f,f^\prime\rangle) = \left\langle \sin(f) , f^\prime \cos(f) \right\rangle$$  |
| $$\cos$$   | $$\cos(\langle f,f^\prime\rangle) = \left\langle \cos(f) , -f^\prime \sin(f) \right\rangle$$ |
| $$\exp$$   | $$\exp(\langle f,f^\prime\rangle) = \left\langle \exp(f) , f^\prime \exp(f) \right\rangle$$  |
| $$\log$$   | $$\log(\langle f,f^\prime\rangle) = \left\langle \log(f) , \frac{f^\prime}{f} \right\rangle \quad (f > 0) $$|
| $$\cdots$$ | $$\cdots$$ |

## Implementation

As we have seen above, for algorithmic differentiation, elementary mathematical operations and functions must be redefined for the dual numbers. This means that the program must be enriched to contain not only its original instructions to compute the function, but also instructions to compute the derivatives.

There are two ways to implement this:

#### Operator overloading

Floats can be replaced by couples of floats representing the dual numbers, and basic operators (and functions) can be redefined. This is called overloading in computer science terms.

Most languages support operator overloading (e.g. Python, R, C++, C#, etc.), but some don't (e.g. Java, Go, Visual Basic), so this option is not always possible.

We will see an example of forward AD with operator overloading in Python towards the end of this blog post.

As we will see, no additional change in the original source code for the function to be differentiated is needed.

#### Source code transformation

Here, the job is done by an AD tool rather than the programmer. The source code of the program is parsed by this AD tool, which then automatically generates the augmented source code that includes the statements for calculating the derivatives along with the original instructions to compute the value of the function.

## Forward AD

Forward AD aims to compute the sensitivity of our function $$F$$ w.r.t. a single direction $$D$$, i.e. the derivatives of all outputs $$(F_1, F_2, \cdots , F_m)$$ with respect to a single input $$x_b$$. In other words, we are computing one column of the Jacobian.

For example, the derivative w.r.t. the first input $$x_1$$, corresponds to direction $$D = (1, 0, \cdots, 0)^T$$.

This is just another way of saying that the program is initialized by assigning a derivative of $$1$$ to the variable of interest (i.e. w.r.t. which we want the derivative to be computed), and a derivative of $$0$$ to all others (since they play the same role as constant in the differentiation).

As it is often the case in the litterature, we'll denote these directional derivatives with a dot on top. The directional derivative is given by:

$$
\dot{Y} = F^\prime(X) \times D = f^\prime_p(X_{p-1}) \times f^\prime_{p-1}(X_{p-2}) \times \cdots \times f^\prime_1(X) \times D
$$

In the formula above, the evaluation is done from right to left, so that each step consists in matrix $$\times$$ vector multiplication (less costly than matrix $$\times$$ matrix multiplications). As a result, the order of execution of the differentiation is the same as the one in which the computation of the function value is done, which makes it possible to carry the computations of both value and derivative in a single (forward) sweep. This is why it is called __forward__ AD.

This hints at simplicity of the code modification needed to make our program capable of computing both $$F$$ and its derivative. For our function, it will look like this:

| Step           | For the value (in the original program) | For the derivative (added to get the enhanced program) |
|----------------|--------------------------------|--------------------------------|
| Initialization | $$X_0 = X$$                    | $$\dot{X}_0 = D$$              |
|----------------|--------------------------------|--------------------------------|
| Step 1         | $$X_1 = f_1(X_0)$$             | $$\dot{X}_1 = f_1^\prime(X_0) \times \dot{X}_0$$ |
|----------------|--------------------------------|--------------------------------|
| ...            | ...                            | ...                            |
|----------------|--------------------------------|--------------------------------|
| Step p         | $$X_p = f_p(X_{p-1})$$         | $$\dot{X}_p = f_p^\prime(X_{p-1}) \times \dot{X}_{p-1}$$ |
|----------------|--------------------------------|--------------------------------|
| Output         | $$Y = X_p$$                    | $$\dot{Y} = \dot{X}_p$$        |
|----------------|--------------------------------|--------------------------------|


## Example in Python

Let's look at an example using operator overloading in Python.

First, we define a class representing dual numbers with a value and a derivative.

```python
class ADfloat(object):
    def __init__(self, value_, derivative_ = 0):
        self.value = value_
        self.derivative = derivative_
```
As we have seen previously, for constants, the derivative is zero. So, we defaulted the derivative attribute to zero in the constructor to simply write `ADfloat(c)` for constants $$c$$.

Second, we need to overload the basic arithmetic operators for this class. In Python this can be achieved as follows:

```python
class ADfloat(object):
    (...)
    # (f + g)' =  f' + g'
    def __add__(self, other):
        return ADfloat(self.value + other.value,
                       self.derivative + other.derivative) 
    
    # (f * g)' =  f' * g + f * g'
    def __mul__(self, other):
        return ADfloat(self.value * other.value,
                       self.derivative * other.value + self.value * other.derivative)

    # (f - g)' =  f' - g' (using the above and the fact that f - g = f + -1. * g)
    def __sub__(self, other):
        return self + (other * ADfloat(-1.))
    
    # (f / g)' = (f * 1/g)' = f' / g - f * g' / g^2
    def __truediv__(self, other):
        return ADfloat(self.value / other.value,
                       self.derivative / other.value - self.value * other.derivative / other.value ** 2)
```
Last, let's define how to print our `ADfloat` objects by implementing the `__str__` function for our class:

```python
class ADfloat(object):
    (...)
    def __str__(self):
        return('({:0.4f}, {:0.4f})'.format(self.value, self.derivative))
```

Now, let's suppose that we want to compute both the value and derivatives of this simple function :

$$
	f(x, y) = \frac{(x+1)\times(x-y)}{x+y+1}
$$

We can define our function as we would do it for floats, but using `ADfloat` objects for constants:

```python
def f(x, y):
    return (x + ADfloat(1.)) * (x - y)/(x + y + ADfloat(1.))
```

Now, let's try to compute the value of the function for $$(x, y) = (3, 2)$$ for example, as well as its derivative w.r.t $$x$$.

```python
x = ADfloat(3., 1.)
y = ADfloat(2., 0.)
print(f(x, y))
#(0.6667, 0.7222)
```
You can check that the value of the function should be $$(\frac{2}{3}, 8) = (0.6666..., 8)$$ and its derivative w.r.t. $$x$$ should be $$(\frac{13}{18}, 8) = (0.72222...., 8)$$, which is in line with what we got above.

Similarly, if we need the derivative w.r.t. $$y$$, we simply set it's derivative to $$1$$ and that of $$x$$ to zero. This leads to:

```python
x = ADfloat(4., 1.)
y = ADfloat(2., 0.)
print(f(x, y))
#(0.6667, -0.7778)
```

This works also exactly in the same way for multidimensional functions, let's define for example:

$$
	g(x, y) = \left(\frac{(x+1)\times(x-y)}{x+y+1}, 2 \times x + y \right)
$$

```python
def g(x, y):
    return [f(x, y), ADfloat(2.) * x + y]
```

We get a (value, derivative) pair for each output:

```python
x = ADfloat(3., 1.)
y = ADfloat(2., 0.)
z = g(x,y)
print(z[0])
#(0.6667, 0.7222)
print(z[1])
#(8.0000, 2.0000)
```

As expected, the value is $$(\frac{2}{3}, 8)$$ and its derivative w.r.t. $$x$$ is $$(\frac{13}{18}, 2)$$.

## AD and approximations
As you might expect, applied to an approximation of a function, the AD will not necessarily give a good approximation of the derivative of the real fonction.
This is not specific to AD, but comes from the underlying math: __the derivative of an approximation is not necessarily a good approximation of the real derivative__.

To illustrate, let's consider the following example, where the exponential function is not available, and is approximated with its Taylor series (truncated to the second degree here to simplify):

$$
\exp(x) = \sum_{k = 0}^{\infty} \frac{x^k}{k!} \approx 1 + x + \frac{x^2}{2}
$$

```python
def approx_exp(x):
    return ADfloat(1.) + x + ADfloat(.5) * x * x
 
print(approx_exp(ADfloat(1.,1.)))
# (2.5, 2.0)
```

With this approximation, we get a derivative at $$1$$ equal to $$2$$, which is not a good approximation (the result is $$e \approx 2.71828...$$).

Exploiting the fact that the derivative of the exponential is the function itself (with the same approximation) would have given $$2.5$$, which is a far better result, and what's more, with zero cost!

This example shows that one should be careful when handling approximations, and not simply blindly apply AD. Indeed, AD shouldn't be an excuse not to think or not to understand your code you are trying to differentiate.

For example, applying AD to some code computing numerically an integral of a function is a bad idea, since you already have the derivative: the function.

## Conclusion

As we have seen in this blog post, forward AD is adapted to getting directional derivatives: derivatives of all the outputs of the function w.r.t. one single input.

If we want to compute the whole Jacobian matrix, forward AD is interesting if the number of outputs is larger than the number of inputs: $$m >> n$$ since it will give the whole matrix in $$n$$ sweeps.

But, what if $$n >> m$$, or if we need only the gradient of a single output of the function?

Actually, this is the most common case. In optimisation for example, we usually have a scalar cost function and we want to determine the set of input values that minimize it. To achieve this, we need the gradient w.r.t. all these inputs. 

Similarly, in finance, we usually have a scalar function (a price, an xVA adjustment, etc.), and we want to compute its sensitivities to a bunch of parameters and market variables.

In such a case, forward AD is not the most efficient option. This is where backward AD or AAD comes into play. It will be the subject of the next blog post. Stay tuned!