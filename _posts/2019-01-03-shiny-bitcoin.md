---
layout: post
title: Shiny Bitcoin
feature-img: "assets/img/pexels/shiny_bitcoin_2.jpg"
tags: [R, Shiny, UI, Widgets, Quandl, Bitcoin]
---

You're good with R? You want to build cool interactive interfaces to share your work with your colleagues (or with the rest of the world) but don't know any HTML/CSS/Javascript?

Look no more, [Shiny](https://shiny.rstudio.com/) is what you need!

In this blog post, we will see how to build a Shiny app, taking this one that shows the evolution of Bitcoin price as an example: [https://youness.Shinyapps.io/BitcoinData/](https://youness.Shinyapps.io/BitcoinData/)

## Shiny

Shiny is an R package developed by the RStudio team and allowing to build web apps in R.
This is particularly interesting thanks to the growing list of [HTML widgets](https://www.htmlwidgets.org) available to use in R.

Shiny apps can be shared to be installed directly on the user's machine (as a package for example), or hosted on a server, but we'll get to that later.

Examples of websites built using Shiny can be found in this [gallery](https://shiny.rstudio.com/gallery/see-more.html).

### Structure of a Shiny app

The structure of a Shiny app is as follows:
```r
ui <- fluidPage(...)
server <- function(input, output){...}
ShinyApp(ui = ui, server = server)
```
It consits of UI (or client) code, that will usually be included in a file called __`ui.R`__, and server code, usually in a filed called __`server.R`__. 

Let's see how each works.

### UI side

This is basically the code that builds the user interface that will be displayed in the web browser.

As its name indicates, the `fluidPage()` function builds a "fuild page". This is a web page (actually an HTML `<div />`) that fills all the available browser width.
Another option is the `fixedPage()` that generates a web page with a fixed width.

This fluid page will contain elements that will be displayed according to some layout. Here, we will keep things simple and use a basic layout featuring:

- a title panel;
- a side bar panel containg some widgets that wil help the user enter some inputs;
- and a main panel to display the results.


```r
ui <- fluidPage(titlePanel(...),
                sidebarPanel(...),
                mainPanel(...)
                )
```


For more on how to make sophisticated layouts, you can refer to this [article](https://Shiny.rstudio.com/articles/layout-guide.html).

Now, let's fill these panels.

#### Input widgets

First, we need some input widgets to get the user's inputs.

Each input widget type can be created using a dedicated function like the following:

| Function          | UI element            |
|-------------------|-----------------------|
|`textInput()`      |Text input control     |
|`passwordInput()`  |Password input control |
|`dateInput()`      |Date input             |
|`fileInput()`      |File upload control    |


Examples of available input widgets (and how they look) can be found [here](https://shiny.rstudio.com/gallery/#widgets).

For this example, the inputs widgets go in the sidebar, and include:

- a slider input to select the date range to cover;
- and a select input (a.k.a. a dropdown list) to select the data to display:

```r
sidebarPanel(
  sliderInput('dates', 'Dates', as.Date('2009-01-03', '%Y-%m-%d'),
              today, dragRange = TRUE, value = c(today - 365, today),
              timeFormat = '%Y-%m-%d'),
  selectInput('data', 'Data', c('BTC/USD', 'Miners Revenue',
                                'Blockchain Size', 'Hash Rate'))
)
```
#### Output functions

Similarly, an output function exists for each output type:

| Function       | Type            |
|----------------|-----------------|
|`htmlOutput()`  |Raw HTML         |
|`imageOutput()` |image            |
|`plotOutput()`  |plot             |
|`tableOutput()` |table            |
|`textOutput()`  |text             |
|`uiOutput()`    |Shiny ui element |
|`plotlyOutput()`|plotly plot      |
| ...            |...              |

In our example, the outputs go in the main panel. They include a plotly plot and some text (caption of the plot, as well as the description of the data being plotted):
```r
mainPanel(
  h3(textOutput('caption')),
  plotlyOutput('plot'),
  h3('Description'),
  textOutput('text')
)
```

The final UI code looks as follows:
```r
library(shiny)

today <- Sys.Date()
fluidPage(
  titlePanel('₿itcoin data'),
  sidebarPanel(
    sliderInput('dates', 'Dates', min = as.Date('2009-01-03', '%Y-%m-%d'), today,
                dragRange = TRUE, value = c(today - 365, today),
                timeFormat = '%Y-%m-%d'),
    selectInput('data', 'Data', c('BTC/USD', 'Miners Revenue',
                                  'Blockchain Size', 'Hash Rate'))
  ),
  mainPanel(
    h3(textOutput('caption')),
    plotlyOutput('plot'),
    h3('Description'),
    textOutput('text')
  )
)
```

### Server side

This is the code that runs on the server side. It transforms the inputs into outputs and renders them.
In this example, it will:

- read the selected choice;
- download the data;
- plot it and update the title and description.

#### Quandl

The Bitcoin data that will be plotted by our Shiny app will be fetched from [Quandl](https://www.quandl.com), a very interesting data provider for classical and alternative data, that has been bought recently by Nasdaq.
What makes it attractive is that it features many data sources, and provides the data in a unified format and using the same API, regardless of its source.

The [`Quandl` R package](https://www.quandl.com/tools/r) is available for free. To be able to do more than 50 calls a day, which will be the case if your website is successfull, you should create an account and set your API key in this way:
```r
Quandl.api_key('<your_Quandl_API_key_here>')
```

<!--
For the plots, we will use [`plotly`](http://plot.ly/r/). Here also, to be able to create charts and publish them online, you should create a free account and set your API key in the following manner:
```r
Sys.setenv("plotly_username" = "<your_plotly_username_here>")
Sys.setenv("plotly_api_key" = "<your_plotly_API_key_here>")
```
-->


#### Accessing the inputs and outputs

For the Shiny app to work, it should be able to access inputs and outputs from the server side.
For this purpose, each input and output has an id that helps identify it on the server's side.

As we have seen previously, the server code is a function of `input` and `output`, inputs can be accessed using their id in this way: `input$<id_of_the_input>`, and the same goes for outputs: `output$<id_of_the_output>`.

#### Reactivity

Because the selected values will change over time, we want our outputs to be reactive to any change in the inputs.

This is achieved by using the inputs in "rendered" outputs. Outputs should be rendered using `render*()` functions. The function to use depends on the output type:

|Output type      | Output using   | Rendered using |
|-----------------|----------------|----------------|
|image            |`imageOutput`   |`renderImage()` |
|plot             |`plotOutput()`  |`renderPlot()`  |
|table            |`tableOutput()` |`renderTable()` |
|text             |`textOutput(`   |`renderText()`  |
|Shiny ui element |`uiOutput()`    |`renderUI()`    |
|plotly plot      |`plotlyOutput()`|`renderPlotly()`|
|...              |...             | ...            |
           
For example, for our [Plotly](http://plot.ly/r/) plot, we can use the following code:

```r
output$plot <- renderPlotly({
    # read the choice, and determine the ticker: 
    ticker <- as.character(dataset_choices$Ticker[dataset_choices$Choice == input$data][1])
    # download data:
    dataset <- Quandl(ticker, start_date = input$dates[1], end_date = input$dates[2])
    # plot using plotly:
    plot_ly(type = 'scatter', mode = 'lines', name = input$data) %>%
        add_trace(x = dataset$Date, y = dataset$Value, showlegend = FALSE)
})
```

It is also possible to create a reactive expression using using the `reactive()` function.
For example, by reading the drop down choice in the following way we can access the value by calling `choice()` throughout the server code of the app.

```r
choice <- reactive(input$data)
```
When the reactive value changes, the functions that use it are notified, and the code will be retriggered to update the text, replot the plot, etc.

Our final server side code is as follows:

```r
library(shiny)
library(Quandl)
library(plotly)

function(input, output) {
  Quandl.api_key('<your_API_key_here>')
  dataset_choices <- data.frame(
    Choice = c('Miners Revenue', 'BTC/USD', 'Blockchain Size', 'Hash Rate'),
    Ticker = c('BCHAIN/MIREV', 'BCHAIN/MKPRU', 'BCHAIN/BLCHS', 'BCHAIN/HRATE'),
    Description = c('Miners revenue in USD (Number of bitcoins mined per day + transaction fees) * market price.',
                    'USD market price of 1 Bitcoin',
                    'The total size of all block headers and transactions. Not including database indexes.',
                    'Estimated number of giga hashes per second (billions of hashes per second) the bitcoin network is performing.'))
  choice <- reactive(input$data)
  output$caption <- renderText(choice())
  output$plot <- renderPlotly({
    ticker <- as.character(dataset_choices$Ticker[dataset_choices$Choice == choice()][1])
    dataset <- Quandl(ticker, start_date = input$dates[1], end_date = input$dates[2])
    plot_ly(type = 'scatter', mode = 'lines', name = choice()) %>%
      add_trace(x = dataset$Date, y = dataset$Value, showlegend = FALSE)
  })
  output$text <- renderText(as.character(
    dataset_choices$Description[dataset_choices$Choice == choice()][1]))
}
```

## Running the app

To run the app locally, simply call the `runApp()` fucntion in your console from R Studio. A local Shiny server will be created, and a pop up will appear showing your app.

A very cool feature is the "showcase mode". To run your app locally in this mode, simply call:

```r
runApp(display.mode = "showcase")
```

This mode is very useful to understand how Shiny apps work, as the code is displayed with the app and the code that is executed is highlighted in realtime as you change the inputs.

## Publishing to ShinyApps.io

When it comes to publishing your Shiny app, the easiest option is to rely on the [ShinyApps.io](https://www.shinyapps.io) plateform.
Creating an account takes 5 minutes, and you can start with a free plan.

The deployment is done using the `rsconnect` package (for **RS**tudio **Connect**) by following these steps:

1. run your app locally;
2. clic on the publish button and follow the instructions to install (or update) any needed packages, authenticate yourself, etc.
3. change your app;
4. run locally;
5. republish;
6. loop...

![Publish button]({{ site.baseurl }}/assets/img/shiny/publish.PNG)

## To go further

- Browse Shiny's official [tutorials](https://shiny.rstudio.com/tutorial/), and [articles](https://shiny.rstudio.com/articles/).
- Learn about [Shiny themes](https://rstudio.github.io/shinythemes/) and CSS to enhance your app's looks.
- Learn about [Shiny proxy](https://www.shinyproxy.io/) and [Shiny server](https://www.rstudio.com/products/shiny/shiny-server/) to deploy your apps on your own...