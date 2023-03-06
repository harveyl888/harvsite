+++
author = "Harvey"
title = "Ordering Pre-selected Items in selectizeInput"
date = "2023-01-27"
description = "Maintaining the order of pre-selected items in selectizeInput"
tags = ["R", "shiny"]
codeMaxLines = 15
draft = false
+++

# SelectizeInput

SelectizeInput is a powerful shiny widget built on the selectize.js library.  The behavior of selectize.js can be extended using plugins, which are available in the `shiny::selectizeInput()` functon.  One particularly useful plugin is "drag_drop" which allows a user to re-order the items selected in a selectizeInput.  When paired with the "remove_button" plugin this makes a powerful UI in which a user may select, reorder and remove items from a list.  
When building a selectizeInput widget, we can list items that have been pre-selected, which allows us to start in a specific state.  With drag-drop and remove_button added, these pre-selected items will appear in the selectizeInput box, already selected.

For example, the code below produces the following output:

{{< figure src="/images/post-images/2023-01-27_selectizeInput_order/selectizeinput_01.png" >}}

```r
shiny::selectizeInput(inputId = "sel", 
                      label = "Selection", 
                      choices = 1:10, 
                      selected = 1:5, 
                          multiple = TRUE, 
                          options = list(
                            plugins= list("remove_button", "drag_drop")
                          ))
```

There is one caveat in this approach which is that the order of selected items is not honored.  For example, the code below produces the following output:


{{< figure src="/images/post-images/2023-01-27_selectizeInput_order/selectizeinput_02.png" >}}

```r
shiny::selectizeInput(inputId = "sel", 
                      label = "Selection", 
                      choices = 1:10, 
                      selected = c(4, 7, 2, 3), 
                          multiple = TRUE, 
                          options = list(
                            plugins= list("remove_button", "drag_drop")
                          ))
```

The correct items are selected but their order has not been preserved.  Instead, they are ordered according to the order of the *choices* parameter.  This is normal behavior and is an artefact in the way that selectize.js works.

# Solution to Order of Pre-selected Items

A solution to this issue lies in re-ordering the *choices* parameter.  The following shiny app will also fail to maintain the order of the selected items:

```r
library(shiny)

vals <- setNames(1:10, sapply(1:10, function(i) paste("Row", i)))
vals_start <- c(4, 7, 2, 3)

ui <- fluidPage(
  fluidRow(
    column(6, uiOutput("ui_selectize")),
    column(6, verbatimTextOutput("ui_selected_values"))
  )
)

server <- function(input, output, session) {
  
  output$ui_selectize <- renderUI({
    shiny::selectizeInput(inputId = "sel", 
                          label = "Selection", 
                          choices = vals, 
                          selected = vals_start, 
                          multiple = TRUE, 
                          options = list(
                            plugins= list("remove_button", "drag_drop")
                          ))
  })
  
  output$ui_selected_values <- renderPrint({
    input$sel
  })
  
}

shinyApp(ui, server)
```

But, with the addition of a line at the start of `output$ui_selectize`, we maintain the order:

```r
library(shiny)

vals <- setNames(1:10, sapply(1:10, function(i) paste("Row", i)))
vals_start <- c(4, 7, 2, 3)

ui <- fluidPage(
  fluidRow(
    column(6, uiOutput("ui_selectize")),
    column(6, verbatimTextOutput("ui_selected_values"))
  )
)

server <- function(input, output, session) {
  
  output$ui_selectize <- renderUI({
    vals <- c(vals[match(vals_start, vals)], vals[which(!vals %in% vals_start)])
    shiny::selectizeInput(inputId = "sel", 
                          label = "Selection", 
                          choices = vals, 
                          selected = vals_start, 
                          multiple = TRUE, 
                          options = list(
                            plugins= list("remove_button", "drag_drop")
                          ))
  })
  
  output$ui_selected_values <- renderPrint({
    input$sel
  })
  
}

shinyApp(ui, server)
```

The additional line `vals <- c(vals[match(vals_start, vals)], vals[which(!vals %in% vals_start)])` simply reorders `vals` so that our selected items come first in their correct order and the other items follow.

```r
vals <- setNames(1:10, sapply(1:10, function(i) paste("Row", i)))
print(vals)
#Row 1  Row 2  Row 3  Row 4  Row 5  Row 6  Row 7  Row 8  Row 9 Row 10 
#    1      2      3      4      5      6      7      8      9     10 

vals_start <- c(4, 7, 2, 3)
vals <- c(vals[match(vals_start, vals)], vals[which(!vals %in% vals_start)])
print(vals)
#Row 4  Row 7  Row 2  Row 3  Row 1  Row 5  Row 6  Row 8  Row 9 Row 10 
#    4      7      2      3      1      5      6      8      9     10 
```

The shiny output now looks like this.  The order of our pre-selected items is honored.

{{< figure src="/images/post-images/2023-01-27_selectizeInput_order/selectizeinput_03.png" >}}
