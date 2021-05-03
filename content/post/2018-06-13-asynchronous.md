+++
author = "Harvey"
title = "Interrupting a background process"
date = "2018-06-13"
description = "Interrupting a background process"
tags = ["R", "shiny"]
+++

I had been running a shiny app in which I could halt a long-running process.  The process would update the shiny app and constantly looked for an updated shiny variable to inform if a cancel button had been pressed.  The code had been adapted from a SO post at 
[https://stackoverflow.com/questions/30587883/is-it-possible-to-stop-executing-of-r-code-inside-shiny-without-stopping-the-sh/34517844#34517844](https://stackoverflow.com/questions/30587883/is-it-possible-to-stop-executing-of-r-code-inside-shiny-without-stopping-the-sh/34517844#34517844).
Unfortunately this utilized the httpuv::service function in a manner for which it was not designed.  At some point, when shiny and httpuv were updated, this functionality ceased to work.  Thanks to some help from Joe Cheng a similar functionality can be derived using an asynchronous function.  A working example is shown below.

```r
library(shiny)

ui <- fluidPage(
  actionButton("start_list", "Start 'list' iterator"),
  actionButton("start_list2", "Start reactive 'list' iterator"),
  actionButton("start_while", "Start 'while' iterator"),
  actionButton("cancel", "Stop")
)

server <- function(input, output, session) {
  
  val <- reactiveValues(b = 100)
  
  
  base_task_iterator <- function(should_continue, iter_body) {
    if (should_continue()) {
      iter_body()
      later::later(~base_task_iterator(should_continue, iter_body))
    }
    invisible()
  }
  
  while_task_iterator <- function(cancelExpr, whileExpr, func) {
      cancelFunc <- rlang::as_function(rlang::enquo(cancelExpr))
      whileFunc <- rlang::as_function(rlang::enquo(whileExpr))
      
      origCancelVal <- isolate(try(silent = TRUE, cancelFunc()))
      cancelled <- function() {
        !identical(origCancelVal, isolate(try(silent = TRUE, cancelFunc())))
      }
      
        base_task_iterator(
        function() {
          !cancelled() && whileFunc()
        },
        func
      )
  }
  
  list_task_iterator <- function(cancelExpr, x, func) {
      cancelExpr <- rlang::enquo(cancelExpr)
      
      origCancelVal <- isolate(try(silent = TRUE, cancelFunc()))
      pos <- 1
      
        while_task_iterator(!!cancelExpr, pos <= length(x), function() {
        i <- pos
        pos <<- pos + 1L
        isolate({
          func(x[[i]])
      })
    })
  }
  
  observeEvent(input$start_list, {
    list_task_iterator(input$cancel, 1:10, function(x) {
      message(x)
      Sys.sleep(1)
    })
  })
  
  observeEvent(input$start_list2, {
    list_task_iterator(input$cancel, 1:10, function(x) {
      val$b <- val$b + 1
      message(val$b)
      Sys.sleep(1)
    })
  })
  
  observeEvent(input$start_while, {
    # Something's wrong with rlang::as_function, I can't use TRUE, only !FALSE
    while_task_iterator(input$cancel, !FALSE, function() {
      message(format(Sys.time()))
      Sys.sleep(1)
    })
  })
}

shinyApp(ui, server)
```