---
layout: post
title: Algorithmic Differentiation - 2. Backward AD
feature-img: "assets/img/pexels/cpu.jpg"
tags: [Python, AD, AAD, Algorithmic differentiation, Operator overloading, Dual numbers]
---

In the [previous blog post](http://byouness.com/2019/01/24/Algorithmic-differentiation-forward-AD.html), we have introduced algorithmic differentiation. We have seen how it works, focusing on its forward mode, and have illustrated with an example in Python relying on operator overloading.

In this post, we tackle the backward mode of algorithmic differentiation. We explain how it works, present its main challenges, and provide a simple example in Python.

## Backward AD

We still have our function $$F$$ that takes some inputs $$X$$, and goes through many intermediary computation steps to turn them into outputs $$Y$$:

$$
Y = F(X) = \left( f_p \circ \cdots \circ f_1 \circ f_0 \right) (X)
$$

This time, however, we want to compute a gradient of one of the outputs w.r.t. all inputs $$(x_1, \cdots, x_n)$$. That is, a row of the Jacobian matrix. We will follow the usual notation and denote it with a bar on top:

$$ \bar{Y} = R \times f^\prime_p(X_{p-1}) \times f^\prime_{p-1}(X_{p-2}) \times \cdots \times f^\prime_1(X)$$

Above, $$R$$ is a row vector giving the output whose gradient we want to compute. For example, for $$F_j(X)$$, $$R$$ should contain $$1$$ in position $$j$$ and zeros elsewhere. In particular, if $$F$$ has a scalar output, then $$R$$ is simply 1.

Remember, $$(X_j)$$ denote the intermediate results:

- $$X_0 = X$$;
- $$X_1 = f_1(X_0)$$;
- $$X_2 = f_2(X_1) = (f_2 \circ f_1)(X_0))$$;
- ...
- $$X_{p-1} = f_{p-1}(X_{p-2}) = (f_{p-1} \circ f_{p-2} \cdots \circ f_1)(X_0)$$.

In the formula giving $$\bar{Y}$$, the computation is done from left to right, meaning that the order of execution of the differentiation is the reverse of the order of computation of the function. This is why this mode is called __backward__ AD.

Each step corresponds to a row $$\times$$ matrix multiplication, which is a transpose or adjoint of a matrix $$\times$$ vector computation, hence the name __Adjoint__ AD or __AAD__ often given to this method.

The reverse orders of computation and differentiation make backward AD more complicated than forward AD. Indeed, this means that the augmented program must imperatively contain two sweeps:

- first, a forward sweep to compute the value of the function;
- then, a backward sweep to compute the gradient.

If we look at our function $$F$$, the computation steps will look like the following:


| Step           | Forward sweep to compute the value (already in the original program) |
|----------------|--------------------------------|
| Initialization | $$X_0 = X$$                    |
|----------------|--------------------------------|
| Step 1         | $$X_1 = f_1(X_0)$$             |
|----------------|--------------------------------|
| ...            | ...                            |
|----------------|--------------------------------|
| Step p         | $$X_p = f_p(X_{p-1})$$         |
|----------------|--------------------------------|
| Output         | $$Y = X_p$$                    |
|----------------|--------------------------------|


| Step           | Backward sweep for the differentiation (added to the enhanced program) |
|----------------|--------------------------------|
| Initialization | $$\bar{X}_p = R$$              |
|----------------|--------------------------------|
| Step 1         | $$\bar{X}_{p-1} = \bar{X}_p \times f^\prime_p(X_{p-1})$$ |
|----------------|--------------------------------|
| ...            | ...                            |
|----------------|--------------------------------|
| Step p         | $$\bar{X}_{0} = \bar{X}_1 \times f^\prime_1(X_{0})$$  |
|----------------|--------------------------------|
| Output         | $$\bar{Y} = \bar{X}_0$$        |
|----------------|--------------------------------|


As we can see from the table above, during the backward sweep, the results of all intermediate steps $$X_j, 1 \leq j \leq p-1$$ are needed.

This adds to the complexity of the AAD as one needs to handle these intermediate results requirement in a smart way. There are two extreme possible approaches:

#### Recompute all?

This option consists in computing everything on the fly during the backward sweep, starting from the input $$X_0$$ and until the needed result is obtained.

For example, at step $$j$$ in the backward sweep, $$X_{j-1}$$ is needed to compute $$\bar{X}_j$$. The program will start from $$X_0$$ and go through steps $$1$$ until $$j$$ of the original program to compute $$X_1$$, then $$X_2$$, ... and until $$X_{j-1}$$ is obtained.

This approach looks the simplest to implement. It also involves a minimal memory consumption as nothing except $$X_0$$ is stored. However, it is very CPU time consuming.

#### Store all?

The "store-all" approach is the complete opposite, all intermediate computation results $$X_j$$ are stored in memory (in what's usually called a "tape") during the forward sweep, so that no intermediate result recomputation is needed during the backward sweep.

As a result, this option is the quickest in terms of CPU time. However, it is harder to implement and can be very memory consuming.

Usually, this second approach is preferable for programs simple enough for memory consumption to remain acceptable. Otherwise, one should seek a tradeoff between this approach and the first where both memory consumption and CPU time are kept to reasonable levels.

For example, rather than storing all intermediate results, one could store only certain "checkpoints": $$(X_{k_1}, X_{k_2}, X_{k_3}, \cdots)$$.

In this case, when a given $$X_j$$ is needed in the backward sweep, if $$j < k_1$$ everything is recomputed starting from the input $$X_0$$. However, if $$k_1 \leq j < k_2$$, then the recomputation starts from the stored $$X_{k_1}$$ value, etc.

## Implementation

Just like forward AD, backward AD could be achieved using source code transformation or operator overloading.

The implementation should handle may aspects:

- storage of intermediate results during the forward swepp to avoid recomputing them as seen above;
- identification of the children of each intermediate results;
- computation of the derivatives during the backward sweep;
- and more for advanced implementations, e.g.: "checkpointing",  simplification of the computation graph when possible, etc.

## Example in Python

Let's look at an example in Python. Just for the forward AD, we will use a new class for our floats, and overload the basic operators for this class. 

As seen above, we need our class to store:

- the variable's `value` during the forward sweep;
- its `children` and the partial derivative w.r.t. each of these children `dvar_dchildren`;
- and its `derivative` that will be computed during the backward sweep.

```python
class AADfloat(object):
    def __init__(self, value_):
        self.value = value_
        self.derivative = None
        self.children = []
        self.dvar_dchildren = []
```

Next, we redefine the basic operators:

```python
class AADfloat(object):
(...)            
    # d(f + g) / df =  d(f + g) / dg = 1.
    def __add__(self, other):
        result = AADfloat(self.value + other.value)
        self.children.append(result)
        other.children.append(result)
        self.dvar_dchildren.append(1.)
        other.dvar_dchildren.append(1.)
        return result
    # d(f - g) / df = 1.
    # d(f - g) / dg = -1. 
    def __sub__(self, other):
        result = AADfloat(self.value - other.value)
        self.children.append(result)
        other.children.append(result)
        self.dvar_dchildren.append(1.)
        other.dvar_dchildren.append(-1.)
        return result
    # d(f * g) / df = g
    # d(f * g) / dg = f 
    def __mul__(self, other):
        result = AADfloat(self.value * other.value)
        self.children.append(result)
        other.children.append(result)
        self.dvar_dchildren.append(other.value)
        other.dvar_dchildren.append(self.value)
        return result
    # d(f / g) / df = 1 / g
    # d(f / g) / dg = -f / g**2 
    def __truediv__(self, other):
        result = AADfloat(self.value / other.value)
        self.children.append(result)
        other.children.append(result)
        self.dvar_dchildren.append(1. / other.value)
        other.dvar_dchildren.append(-self.value / other.value ** 2)
        return result
```
Each operator does three things:

1. it computes the value as the sum of the two values;
2. it registers itself as a child of both operands;
3. it computes the derivative w.r.t. each of the operands.

For example, for addition these detivatives are 1 because:

$$\frac{\partial(x+y)}{\partial x} = \frac{\partial(x+y)}{\partial y} = 1$$

Similarly, for multiplication:

$$
\frac{\partial(x \times y)}{\partial x} = y , \frac{\partial(x \times y)}{\partial y} = x
$$

and for division:

$$
\frac{\partial \left(\frac{x}{y} \right)}{\partial x} = \frac{1}{y} ,
\frac{\partial \left(\frac{x}{y} \right)}{\partial y} = \frac{-x}{y^2}
$$

Now, we have everything we need to compute the gradient, all that's left is to define the function that performs the computation:

```python
class AADfloat(object):
	(...)
    def compute_derivative(self):
        if self.derivative is None:
            self.derivative = sum(dv_dc * c.compute_derivative()
                                  for dv_dc, c in zip(self.dvar_dchildren, self.children))
        return self.derivative
```

This function replaces the variable $$v$$ by its children $$(c_1, c_2, ...)$$ in this way:

$$
	\frac{\partial v}{\partial x} = \sum_i \frac{\partial v}{\partial c_i} \times \frac{\partial c_i}{\partial x}
$$

In the code block above, the first term in the product corresponds to the `dv_dc` term, while the second corresponds to `c.compute_derivative()`, which will compute the derivative of each child in the same way, recursively.

Last, we redefine the `__str__` function for our class, to print the value and derivative in a nice way:

```python
class AADfloat(object):
	(...)
    def __str__(self):
        aad_float_str = 'Value = {:0.4f}'.format(self.value)
        if self.derivative is None:
            aad_float_str += ', the backward sweep has not been performed.'
        else:
            aad_float_str += ', Derivative = {:0.4f}.'.format(self.derivative)
        return(aad_float_str)
```

Let's consider the same example of the previous post:

$$
f(x, y) = \frac{(x + 1)\times(x - y)}{x + y + 1}
$$

and evaluate its value and gradient for $$(x, y) = (3, 2)$$.

First, we compute the intermediate steps leading to the value (forward sweep):

```python
# Forward sweep
x = AADfloat(3.)
y = AADfloat(2.)
a = x + AADfloat(1.)
b = x - y
c = a * b
d = a + y
z = c / d
```

We can see that the intermediate computation steps values have been stored:
```python
print(a)
# Value = 4.0000, the backward sweep has not been performed.
print(b)
# Value = 1.0000, the backward sweep has not been performed.
print(c)
# Value = 4.0000, the backward sweep has not been performed.
print(d)
# Value = 6.0000, the backward sweep has not been performed.
```

and that the children of each variable are correctly registered, for exemple:
```python
x.children == [a, b]
# True
y.children == [b, d]
# True
```

Next, we initialize the derivative of the output w.r.t itself is 1: $$\frac{\partial z}{\partial z} = 1$$ , and compute the derivatives (backward sweep):

```python
# Backward sweep
z.derivative = 1.
x.compute_derivative()
y.compute_derivative()
```

We obtain the same values as in the previous post for the gradient:
$$
\begin{aligned}
  \nabla f (3,2) & = \left( \frac{\partial f}{\partial x}(3,2), \frac{\partial f}{\partial y}(3,2) \right) \\
   & = \left(\frac{13}{18}, \frac{-7}{9} \right) \approx (0.7222, -0.7778)
\end{aligned}
$$

```python
print(x)
# Value = 3.0000, Derivative = 0.7222.
print(y)
# Value = 2.0000, Derivative = -0.7778.
```

## Conclusion

We have seen how backward or adjoint AD works, what are its implementation challenges, and illustrated with a very simple example in Python.