+++
author = "Harvey"
title = "Shiny Reactivity"
date = "2021-06-02"
description = "List of Reactives vs Reactive List"
tags = ["R", "shiny", "Shiny modules"]
draft = false
codeMaxLines = 70
+++

Examples of passing a list of reactives, reactiveValues list and reactiveValues to a shiny module.  This helps explain reactivity when passing values to a shiny module.  In each case we pass a `value A` which triggers `mod A` and `value B` which triggers `mod B`.  The code below and output demonstrates which outputs in the shiny module are updated upon changing a value in the main app.  

-  Example 1 - list of reactives.  Two reactives passed to shiny module in a list.  When `value A` is updated only `mod A` updates.  
-  Example 2 - ReactiveValues List.  Two reactives passed to shiny module in a reactive list.  The individual components are not reactive themselves but the list is.  This means that when one list item is changed it triggers updates for each output that implements any member of the list.  When `value A` is updated, both `mod A` and `mod B` update.  
-  Example 3 - ReactiveValues.  Two reactiveValues passed to shiny module.  When `value A` is updated only `mod A` updates.


## Example 1 - List of Reactives

Pass two reactives (`react_A` and `react_B`) to a shiny module.  `react_A` is attached to a reactiveVal and initially set to **A**.  `react_B` is a reactive set to **B**.  `react_A` is then changed to **C** by changing the reactiveVal.  
In this setup `react_A` and `react_B` are sent to the shiny module when it is called.  `react_A` is updated which triggers an update of the module.

Output to the console is:
```r
[1] "response from module A: A"
[1] "response from module B: B"
[1] "response from module A: C"
```

```r
## checking reactivity - reactive list vs list of reactives
## list of reactives
library(shiny)

## module UI
mod_UI <- function(id) {
  ns <- NS(id)
  tagList(
    uiOutput(ns("modA")),
    uiOutput(ns("modB"))
  )
}

## module server
mod <- function(id, inputvar) {
  moduleServer(
    id,
    function(input, output, session) {
      output$modA <- renderUI({
        req(inputvar$A())
        print(paste0("response from module A: ", inputvar$A()))
        h3(paste0("response from module A: ", inputvar$A()))
      })
      
      output$modB <- renderUI({
        req(inputvar$B())
        print(paste0("response from module B: ", inputvar$B()))
        h3(paste0("response from module B: ", inputvar$B()))
      })
    }
  )
}

ui <- fluidPage(

  mod_UI("shinymod")

)

server <- function(input, output, session) {

  rv <- reactiveVal("A")

  react_A <- reactive({
    rv()
  })

  react_B <- reactive({
    "B"
  })

  mod("shinymod", inputvar = list(A = reactive(react_A()), B = reactive(react_B())))

  ## Update rv$A triggers refresh of modA in shiny module but not modB
  observe({
    rv("C")
  })

}

shinyApp(ui, server)
```

## Example 2 - ReactiveValues List

Pass a reactiveValue, `rv$AB` containing a list of two members (**A** = A and **B** = B) to a shiny module.  One member of `rv$AB` is then changed so that `rv$AB` contains **A** = C and **B** = B.  
In this setup `rv$AB` is sent to the shiny module when it is called.  `rv$AB` is updated which triggers an update of the module.

Output to the console is:
```r
[1] "response from module A: A"
[1] "response from module B: B"
[1] "response from module A: C"
[1] "response from module B: B"
```

Here it is clear that updating the reactiveValue triggers two updates - one for the first list member (which changed) and one for the second (which did not).  This is a highly inefficient way of passing data to a module.

```r
## checking reactivity - reactive list vs list of reactives
## reactiveValue
library(shiny)

## module UI
mod_UI <- function(id) {
  ns <- NS(id)
  tagList(
    uiOutput(ns("modA")),
    uiOutput(ns("modB"))
  )
}

## module server
mod <- function(id, inputvar) {
  moduleServer(
    id,
    function(input, output, session) {
      
      output$modA <- renderUI({
        req(inputvar()$A)
        print(paste0("response from module A: ", inputvar()$A))
        h3(paste0("response from module A: ", inputvar()$A))
      })
      
      output$modB <- renderUI({
        req(inputvar()$B)
        print(paste0("response from module B: ", inputvar()$B))
        h3(paste0("response from module B: ", inputvar()$B))
      })
  
    }
  )
}


ui <- fluidPage(

  mod_UI("shinymod")

)

server <- function(input, output, session) {

  rv <- reactiveValues(
    AB = list()
  )

  observe({
    rv$AB <- list(A = "A", B = "B")
  })

  mod("shinymod", inputvar = reactive(rv$AB))

  ## Update rv$AB triggers refresh of modA and modB in shiny module
  observe({
    rv$AB <- list(A = "C", B = "B")
  })

}

shinyApp(ui, server)
```

## Example 3 - ReactiveValues

Pass a set of reactiveValues, `rv` containing two elements (**A** = A and **B** = B) to a shiny module.  Change one element (**A** = C).
In this setup `rv` is sent to the shiny module when it is called.  `rv` is updated which triggers an update of the module.

Output to the console is:
```r
[1] "response from module A: A"
[1] "response from module B: B"
[1] "response from module A: C"
```

```r
## checking reactivity - reactive list vs list of reactives
## reactiveValues
library(shiny)

mod_UI <- function(id) {
  ns <- NS(id)
  tagList(
    uiOutput(ns("modA")),
    uiOutput(ns("modB"))
  )
}

## module server
mod <- function(id, inputvar) {
  moduleServer(
    id,
    function(input, output, session) {
      
      output$modA <- renderUI({
        req(inputvar()$A)
        print(paste0("response from module A: ", inputvar()$A))
        h3(paste0("response from module A: ", inputvar()$A))
      })
      
      output$modB <- renderUI({
        req(inputvar()$B)
        print(paste0("response from module B: ", inputvar()$B))
        h3(paste0("response from module B: ", inputvar()$B))
      })
  
    }
  )
}

ui <- fluidPage(

  mod_UI("shinymod")

)

server <- function(input, output, session) {

  rv <- reactiveValues(
    A = "A",
    B = "B"
  )

  mod("shinymod", inputvar = reactive(rv))

  ## Update rv$A triggers refresh of modA in shiny module
  observe({
    rv$A <- "C"
  })

  ## Update rv$B does not trigger refresh of modB in shiny module (no change to rv$B)
  observe({
    rv$B <- "B"
  })

}

shinyApp(ui, server)
```