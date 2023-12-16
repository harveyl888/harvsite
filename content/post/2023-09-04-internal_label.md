+++
author = "Harvey"
title = "Internal labels in textInput box"
date = "2023-09-04"
description = "Moving a shiny textInput label to inside the textbox"
tags = ["R", "shiny"]
codeMaxLines = 15
draft = true
+++

# Shiny textInput label

This is just a short post highlighing how you can move a textInput label so that it is within the textbox textInput box itself.  This is a style that can be used when you have multiple textInputs and wish to preserve screen height.

First we define a css class to reduce the size of the label text and reposition it relative to the text box.

```css
.internal-label .control-label {
  position: relative;
  display: inline-block;
  width: 100%;
  top: 20px;
  right: 5px;
  text-align: right;
  font-size: 50%;
  color: #888;
  z-index: 2;
}
```

Then add the class to a textInput widget to move the label.

```r
library(shiny)

ui <- fluidPage(
  tags$head(tags$link(rel = "stylesheet", type = "text/css", href = "style.css")),
  br(),
  fluidRow(column(6, offset = 1, textInput("txt1", label = "Original Textbox Label", value = "Textbox Text"))),
  fluidRow(column(6, offset = 1, textInput("txt1", label = "Internal Textbox Label", value = "Textbox Text") |>
      tagAppendAttributes(class = "internal-label")))
)

server <- function(input, output, session) {}

shinyApp(ui, server)
```

{{< figure src="/images/post-images/2023-09-04-textinput_label/textinput_label.png" >}}
