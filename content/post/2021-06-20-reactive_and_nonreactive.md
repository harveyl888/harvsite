+++
author = "Harvey"
title = "Passing a Mixture of Reactive and Non-Reactives to a Shiny Module"
date = "2021-06-20"
description = "Passing a Mixture of Reactive and Non-Reactives to a Shiny Module"
tags = ["R", "shiny"]
draft = false
+++

Generally there is no issue in sending a list of parameters (reactive and non-reactive) to a shiny module.  Here's an example where a shiny module would be called multiple times, programatically, where the reactive nature of the parameters may be variable (reactive in one instance but not in another).  One way to deal with this is to read in the list of parameters and convert the non-reactive ones to reactive.  Those originally reactive, remain so and therefore update on a change.  In the examples below a single reactive or non-reactive list is passed from the parent shiny app to a shiny module.  The concept works for a single or multiple inputs.

## Example 1 - this shiny module will fail

This version of the shiny module fill fail - it expects a reactive but is passed a non-reactive.

```r
library(shiny)

appmod_UI <- function(id) {
  ns <- NS(id)
  verbatimTextOutput(ns("txt"))
}

appmod <- function(input, output, session, inputlist) {

  output$txt <- renderPrint({
    message("Updating renderPrint")
    print(inputlist()[["val1"]])
    print(inputlist()[["val2"]])
  })

}

ui <- fluidPage(
  appmod_UI("mod1")
)

server <- function(input, output, session) {

  callModule(appmod, "mod1", inputlist = list(val1 = "value 1", val2 = "value 2"))

}

shinyApp(ui, server)
```

## Example 2 - reacive expected and reactive passed

This is a simple example of a shiny module that will work.  It expects a reactive which is exactly what is passed to the module.

```r
library(shiny)

appmod_UI <- function(id) {
  ns <- NS(id)
  verbatimTextOutput(ns("txt"))
}

appmod <- function(input, output, session, inputlist) {

  message("Starting shiny module")

  if (is.reactive(inputlist)) {
    message("Returning inputlist")
    inputlist_1 <- inputlist
  } else {
    message("Returning reactive inputlist")
    inputlist_1 <- reactive(inputlist)
  }

  output$txt <- renderPrint({
    message("Updating renderPrint")
    print(inputlist_1()[["val1"]])
    print(inputlist_1()[["val2"]])
  })

}

ui <- fluidPage(
  appmod_UI("mod1")
)

server <- function(input, output, session) {

  callModule(appmod, "mod1", inputlist = reactive(list(val1 = "value 1", val2 = "value 2")))

}

shinyApp(ui, server)

```

## Example 3 - reacive expected and non-reactive passed

Another example of a shiny module that will work.  In this case the non-reactive passed to the shiny module is converted into a reactive.

```r
library(shiny)

appmod_UI <- function(id) {
  ns <- NS(id)
  verbatimTextOutput(ns("txt"))
}

appmod <- function(input, output, session, inputlist) {

  message("Starting shiny module")

  if (is.reactive(inputlist)) {
    message("Returning inputlist")
    inputlist_1 <- inputlist
  } else {
    message("Returning reactive inputlist")
    inputlist_1 <- reactive(inputlist)
  }

  output$txt <- renderPrint({
    message("Updating renderPrint")
    print(inputlist_1()[["val1"]])
    print(inputlist_1()[["val2"]])
  })
}

ui <- fluidPage(
  appmod_UI("mod1")
)

server <- function(input, output, session) {

  callModule(appmod, "mod1", inputlist = list(val1 = "value 1", val2 = "value 2"))

}

shinyApp(ui, server)

```

## Example 4 - reacive expected and non-reactive passed

Final example of a shiny module that will work.  In this case a reactive is passed and then changed (forcing an update in the shiny module).

```r
library(shiny)

appmod_UI <- function(id) {
  ns <- NS(id)
  verbatimTextOutput(ns("txt"))
}

appmod <- function(input, output, session, inputlist) {

  message("Starting shiny module")

  if (is.reactive(inputlist)) {
    message("Returning inputlist")
    inputlist_1 <- inputlist
  } else {
    message("Returning reactive inputlist")
    inputlist_1 <- reactive(inputlist)
  }

  output$txt <- renderPrint({
    message("Updating renderPrint")
    print(inputlist_1()[["val1"]])
    print(inputlist_1()[["val2"]])
  })


}

ui <- fluidPage(
  appmod_UI("mod1"),
  actionButton("but_update", "Update Values")
)

server <- function(input, output, session) {

  rv <- reactiveValues(
    i = list()
  )

  observe({
    message("Define initial values")
    rv$i <- list(val1 = "value 1", val2 = "value 2")
  })

  callModule(appmod, "mod1", inputlist = reactive(rv$i))

  observeEvent(input$but_update, {
    message("Button press")
    rv$i[["val1"]] <- "value 3"
  })

}

shinyApp(ui, server)

```