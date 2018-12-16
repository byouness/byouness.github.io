---
layout: post
title: Interactive plots in Jupyter Notebooks with IpyWidgets
feature-img: "assets/img/ipywidgets/jupiter2.jpg"
tags: [Python, Jupyter notebook, Matplotlib, Plotly, options]
---

In this blog post, we will see how to add interactive plots in Jupyter notebooks using Ipy widgets.

As an example, we will create an interactive options strategy payoff displayer using Ipywidgets, Matplotlib and Plotly.

## Jupyter Notebooks

Jupyter notebooks are documents combining three elements:

  - Text in markdown format;
  - Computer code that can be modified and executed interactively;
  - and other elements such as widgets, math equations, etc.

These documents are in JSON format. They are served by the [Jupyter notebook app](https://jupyter.org/) on the web browser as a webpage.

Examples of notebooks (in static mode) are available in the [nbviewer page](https://nbviewer.jupyter.org/). Here is an [example](https://nbviewer.jupyter.org/github/fonnesbeck/statistical-analysis-python-tutorial/blob/master/3.%20Plotting%20and%20Visualization.ipynb).

Jupyter notebooks are very widely used by data scientists for exploratory analysis, interactive programming and are a fantastic means to share reproducible research.

Initially, only Python was supported, but the [list of supported languages](https://github.com/jupyter/jupyter/wiki/Jupyter-kernels) keeps growing. It already includes [R](https://github.com/IRkernel/IRkernel), [Matlab](https://github.com/calysto/matlab_kernel), [Octave](https://github.com/calysto/octave_kernel), [Julia](https://github.com/JuliaLang/IJulia.jl) and many others

## IPyWidgets

In addition to interactive code cells that can be edited and executed, another very fun way to interact with Jupyter notebooks is using [Ipywidgets](https://ipywidgets.readthedocs.io/en/latest/).

Unfortunately though, these widgets are available only for the Python kernel (at the time of writing). Some work has been done towards making the widgets available to the other Kernels (e.g. https://github.com/jupyter-widgets/ipywidgets/issues/41) but we aren't there yet.

Another blog post might follow for R alternatives to Jupyter Notebooks, such as:

- [R Notebooks](https://rmarkdown.rstudio.com/r_notebooks.html);
- [shiny apps](http://shiny.rstudio.com/);
- and [shiny dashboards](https://rstudio.github.io/shinydashboard/index.html).

For this one, let's stick to Python.  At the end of this post, we will have built a Notebook with an interactive options strategy displayer.

## Hello world!

Let's start with our first example. We define a function that takes a name as an argument and simply says hello:

```python
import ipywidgets as wdgts
def hello(name):
    return 'Hello ' + name + '!'
wdgts.interact(hello, name = 'world')
```

![hello world example]({{ site.baseurl }}/assets/img/ipywidgets/hello_world.PNG)

This will display a text area, initially containing the string `'world'`. Whenever this string is modified, the function `hello()` will be called with the new string as an argument.

What if we don't want to say hello to everybody, but only to a list of friends? 
Easy, we can list their names in a drop-down list:

```python
friends = ['Bugs', 'Daffy', 'Droopy', 'Tom', 'Jerry']
wdgts.interact(hello, name = friends)
```

![hello friends]({{ site.baseurl }}/assets/img/ipywidgets/hello_world_2.PNG)

This example already shows how fun these widgets can be. Let's spice things up and combine them with charts 

## Options payoffs with Matplotlib

Let's build a payoff displayer for european options, starting with a single option. We need a function giving us the payoff of vanilla options, let's call it `payoff()`;
```python
import numpy as np
def payoff(underlying, direction, option, strike):
    z = 1 if option == 'call' else -1
    e = 1 if direction == 'long' else -1
    return e * np.maximum(z * (underlying - strike), 0)
```
And another one that plots the payoff for underlying values ranging between 0 and 200 using Matplotlib.
```python
%matplotlib notebook
import matplotlib.pyplot as plt
def plot_payoff(direction, option, strike):
    x = np.linspace(0, 200, num = 1000)
    plt.figure()
    plt.plot(x, payoff(x, direction, option, strike))
    plt.show()
```

Above, we used the Jupyter magic function `%matplotlib notebook` in order to set up Matplotlib to be used with the notebook backend (see [here](http://ipython.readthedocs.io/en/stable/interactive/plotting.html) for more info on matplotlib backends).

Now, we simply call the `interact()` function on our plotting function, like we did with the hello world example:

```python
import ipywidgets as wdgts
wdgts.interact(plot_payoff, direction = ['long', 'short'],
               option = ['call', 'put'], strike = c(0, 200))
```
 
Playing with the various combination, we can draw the vanilla options payoff profiles that are found at the beginning of any options textbook. For example, the payoff of a long call is displayed below:

![Long call payoff]({{ site.baseurl }}/assets/img/ipywidgets/long_call.PNG)

## Options strategies with Plotly

In some circumstances, you might want to trigger the update manually. For example, if the code to be called whenever the widget is updated is too heavy.

This is possible using the `interact_manual()` function instead of `interact()`. With this function, a button is added to the interact controls and the code is trigger only upon clicking this button.

To illustrate, we build on the last example and rather than a single option, we display the payoff of a strategy involving 2 or 3 options.

We will use Plotly, which is a powerful plotting library that enables to create interactive graphs where you can zoom-in, zoom-out, click on a serie in the legend to display or hide it, etc.
let's modify our plotting function to involve three options with various quantities, and switch to Plotly instead of Matplotlib.

To be able to use Plotly online, one needs to create an account and get an API key. To use it in a Notebook just locally, the offline mode is enough: 

```python
import plotly.offline as pltly
import plotly.graph_objs as go
pltly.init_notebook_mode(connected = True)
```

Our plotting function now looks like the following, the plot is constructed using the Plotly's `Scatter` function, and is displayed using the `iplot()` function.

```python
def plot_strategy_payoff(quantity_1, direction_1, option_1, strike_1,
                         quantity_2, direction_2, option_2, strike_2,
                         quantity_3, direction_3, option_3, strike_3):
    quantities = [quantity_1, quantity_2, quantity_3]
    directions = [direction_1, direction_2, direction_3]
    options = [option_1, option_2, option_3]
    strikes = [strike_1, strike_2, strike_3]
    x = np.linspace(0, 200, num = 1000)
    y = 0.0
    plot_data = []
    for i in range(3):
        if quantities[i] != 0:
            y_i = quantities[i] * payoff(x, directions[i], options[i], strikes[i])
            plot_data += [go.Scatter(x = x, y = y_i, mode = 'line',
                                     line = dict(dash = 'dash'),
                                     name = 'option' + str(i+1))]
            y += y_i
    plot_data += [go.Scatter(x = x, y = y, mode = 'line', name = 'strategy')]
    pltly.iplot(plot_data)
```

We now define our widgets and call the `interact_manual()` function.

Here, we define and pass the widgets to the function instead of lists, tuples, etc. so that we can customize their description text:

```python    
wdgts.interact_manual(plot_strategy_payoff,
                      quantity_1 = wdgts.IntSlider(1, 0, 10, description = 'Option 1: Quantity'),
                      direction_1 = wdgts.Dropdown(options = ['long', 'short'],
                                                   value = 'long', description = 'Direction'),
                      option_1 = wdgts.Dropdown(options = ['call', 'put'],
                                                value = 'call', description = 'Option'),
                      strike_1 = wdgts.IntSlider(80, 0, 200, description = 'Strike'),
                      quantity_2 = wdgts.IntSlider(1, 0, 10, description = 'Option 2: Quantity'),
                      direction_2 = wdgts.Dropdown(options = ['long', 'short'],
                                                   value = 'long', description = 'Direction'),
                      option_2 = wdgts.Dropdown(options = ['call', 'put'],
                                                value = 'call', description = 'Option'),
                      strike_2 = wdgts.IntSlider(120, 0, 200, description = 'Strike'),
                      quantity_3 = wdgts.IntSlider(2, 0, 10, description = 'Option 3: Quantity'),
                      direction_3 = wdgts.Dropdown(options = ['long', 'short'],
                                                   value = 'short', description = 'Direction'),
                      option_3 = wdgts.Dropdown(options = ['call', 'put'],
                                                value = 'call', description = 'Option'),
                      strike_3 = wdgts.IntSlider(100, 0, 200, description = 'Strike'))
``` 

Now simply click on the \[Run interact\] button, and you will get a beautiful butterfly strategy interactive plot:

![Butterfly strategy plotly plot]({{ site.baseurl }}/assets/img/ipywidgets/butterfly.PNG)

Now, you can play around to see whether you know your options strategies. See if you can draw a straddle, a strangle, bulls and bear spreads using calls or using puts, etc.