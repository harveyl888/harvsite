+++
author = "Harvey"
title = "Resizable bslib Cards"
date = "2024-05-01"
description = "bslib dashboards with resizable cards"
tags = ["R", "shiny"]
codeMaxLines = 15
draft = true
+++

This is a simple example of resizing {bslib} cards for a dashboard.  A card can be made resizable (vertical, horizontal or both) by using the `resize` property.  In order to use this in a {bslib} dashboard it can be added using `tags$style` or included in `bs_theme()`.  Adding the `resize-vertical` class adds a resize anchor to the botton right of a card.

```r
library(bslib)

ui <- page_fillable(
  tags$head(
    tags$style("
      .resize-vertical {
        resize: vertical;
      }
    ")
  ),
  layout_sidebar(
    sidebar = sidebar(
      width = 250,
      h3("controls")
    ),
    card(
      card_title("Card 1"),
      height = "60%",
      fill = FALSE,
      class = 'resize-vertical',
      card_body(
        plotOutput('plt')
      )
    ),
    layout_column_wrap(
      card(
        card_title("Card 2"),
        card_body(
        )
      ),
      card(
        card_title("Card 3"),
        card_body()
      )
    )
  )
)

server <- function(input, output, session) {
  output$plt <- renderPlot({
    plot(mtcars$mpg, mtcars$hp)
  })
  output$data <- renderDataTable(mtcars)
}

shinyApp(ui, server)
```
