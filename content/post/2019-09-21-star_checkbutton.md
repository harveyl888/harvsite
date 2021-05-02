+++
author = "Harvey"
title = "Star Checkbutton"
date = "2019-09-21"
description = "Icon star as a checkbox in a shiny app"
tags = ["R", "shiny"]
+++

Here's a short shiny app along with some css that allows a font-awesome icon to behave as a checkbox.  The font-weight setting in the css file determines if the icon should be solid (font-weight = 900 for `TRUE`) or regular (font-weight = 400 for `FALSE`).

![](/images/post-images/2019-09-21-star_checkbutton/star_checkbox.png)

### checkbox.css
```css
.fancy-checkbox input[type="checkbox"] {
    display: none;
}
 
.fancy-checkbox span:before {
    content: "\f005"; 
    font-family: "Font Awesome 5 Free";
    font-size: 1.5em;
    font-style: normal;
    font-weight: 400;
    text-decoration: inherit;
}
 
.fancy-checkbox input[type="checkbox"]:checked ~ span:before {
    font-weight: 900;
}
```

### app.R

```r
library(shiny)

checkboxbutton <- function(inputId, value = FALSE, color = "#000000") {
  
  if (value) {
    checked_str <- 'checked="checked"'
  } else {
    checked_str <- NULL
  }
  inputTag <- HTML(paste0('<input id="', inputId, '" type = "checkbox" ', checked_str, '" />'))
  tags$div(class="checkbox",
           tags$label(NULL, class="fancy-checkbox", style = paste0("color: ", color),
                      inputTag,
                      tags$span()
                      )
           )
}

server <- function(input, output, session) {
  
  output$txt1 <- renderPrint(
    input$chk1
  )
  
}

ui <- fluidPage(
  tags$head(
    tags$link(rel = "stylesheet", type = "text/css", href = "https://cdnjs.cloudflare.com/ajax/libs/font-awesome/5.3.1/css/all.min.css"),
    tags$link(rel = "stylesheet", type = "text/css", href = "checkbox.css")
  ),
  br(),
  fluidRow(
    column(1, checkboxbutton('chk1', color = "darkred")),
    column(2, verbatimTextOutput('txt1'))
  )
)

shinyApp(ui, server)
```
