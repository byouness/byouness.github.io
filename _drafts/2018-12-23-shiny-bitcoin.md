---
layout: post
title: My first Shiny website
feature-img: "assets/img/ipywidgets/jupiter2.jpg"
tags: [R, Shiny, Quandl, Bitcoin]
---

In this blog post, we will see how to set up a Shiny website. Given the recent news on Bitcoin price decline, we will take a website showing some info about Bitcoin as an example.

In this example, we will also use Quandl, which is a very interesting data provider which has been bought recently by Nasdaq.


## Shiny

## Nasdaq

## Application


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