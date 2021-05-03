+++
author = "Harvey"
title = "Using CSS Variables with Shiny"
date = "2018-07-13"
description = "How to change a CSS variable in a shiny app"
tags = ["R", "shiny", "javascript"]
+++

# Method One - Does not work with IE

CSS variables are a powerful way to make wholesale changes to the style throughout an entire document.  They are set using a custom notation `--variable_name` and accessed throughout the document using `var(--variable_name)`.  With a little javascript these variables can be used in Shiny apps and updated as needed.  The example below contains three files: a shiny app (app.R), the CSS styling (style.css) and the associated javascript functions which update the CSS variables (var\_change.js).  The CSS and javascript files are stored in the www folder.  This method works well under Chrome but does not work with Internet Explorer (css variables do not work under IE).  Later on in this post we'll see how to perform the same wholesale changes but with IE.

### app.R

```r
library(shiny)

intial_size <- 20
intial_color <- 'orange'

server <- function(input, output, session) {
  
  session$sendCustomMessage("col_change", intial_color)
  session$sendCustomMessage("size_change", intial_size)
  
  observeEvent(input$but1, {
    session$sendCustomMessage("col_change", "green")
  })
  
  observe({
    session$sendCustomMessage("size_change", input$sld1)
  })
}

ui <- fluidPage(
  tags$head(
    tags$link(rel = "stylesheet", type = "text/css", href = "style.css"),
    tags$script(src="var_change.js")
  ),
  actionButton('but1', 'change color'),
  sliderInput('sld1', 'change size', value = 20, min = 1, max = 100, step = 1),
  h1("Heading 1"),
  h2("Heading 2"),
  h3("Heading 3")
)

shinyApp(ui, server)
```

### style.css
```css
:root {
  --heading-color: black;
  --heading-size: 12px;
}

h1 {
  color: var(--heading-color);
  font-size: var(--heading-size);
}

h2 {
  color: var(--heading-color);
}

h3 {
  color: var(--heading-color);
}
```

### var\_change.js
```javascript
Shiny.addCustomMessageHandler("col_change", col_change);
Shiny.addCustomMessageHandler("size_change", size_change);

function col_change(x) {
  document.documentElement.style.setProperty('--heading-color', x);
}

function size_change(x) {
  document.documentElement.style.setProperty('--heading-size', x + "px");
}
```

# Method Two - Not as elegant but does work with IE

### app.R

```r
library(shiny)

intial_size <- 20
intial_color <- 'orange'

server <- function(input, output, session) {
  
  session$sendCustomMessage("col_change", intial_color)
  session$sendCustomMessage("size_change", intial_size)
  
  observeEvent(input$but1, {
    session$sendCustomMessage("col_change", "green")
  })
  
  observe({
    session$sendCustomMessage("size_change", input$sld1)
  })
}

ui <- fluidPage(
  tags$head(
    tags$link(rel = "stylesheet", type = "text/css", href = "style.css"),
    tags$script(src="var_change.js")
  ),
  actionButton('but1', 'change color'),
  sliderInput('sld1', 'change size', value = 20, min = 1, max = 100, step = 1),
  h1("Heading 1", class = "header_col_change header_size_change"),
  h2("Heading 2", class = "header_col_change"),
  h3("Heading 3", class = "header_col_change")
)

shinyApp(ui, server)
```

### style.css
```css

header_col_change {
  color: black;
}

header_size_change {
  font-size: 12;
}
```

### var\_change.js
```javascript
Shiny.addCustomMessageHandler("col_change", col_change);
Shiny.addCustomMessageHandler("size_change", size_change);

function col_change(x) {
  $(".header_col_change").css('color', x)
}

function size_change(x) {
  $(".header_size_change").css('fontSize', x)
}
```
