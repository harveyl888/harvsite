+++
author = "Harvey"
title = "Sortable Tables"
date = "2017-11-28"
description = "Sorting rows in a datatable"
tags = ["R", "shiny", "javascript"]
+++

The datatable widget is a powerful table-building tool used to display data in shiny apps.  It can use many of the datatable plugins and extensions but a small javascript hack is needed for the rowReorder extension which allows rows to be reordered using drag and drop.

In the example below, the rowReorder options are set to selector = 'tr' (select by clicking anywhere on a row) and update = FALSE (do not fire an update to rowReorder).  The removal of the update is due to the fact that the rows are not reordered properly in a shiny app.  We can now use the row-order event which is fired irrespective of the setting of the update option (row-reordered, however, is only fired when update = true).  The javascript function called on row-reorder simply grabs the new order of the table rows and exports it to a shiny variable.  This can then be used to update the table.

```r
library(shiny)
library(DT)

## Sortable table

server <- function(input, output) {
  
  df <- reactiveValues(data = mtcars[c(1:20), c('mpg', 'cyl', 'disp')],
                       data_reordered = mtcars[c(1:20), c('mpg', 'cyl', 'disp')])

  output$tab1 <- DT::renderDataTable({
    DT::datatable(df$data, 
                  selection = 'none', 
                  extensions = c('Scroller', 'RowReorder'), 
                  options = list(rowReorder = list(selector = 'tr', update = FALSE), 
                                 columnDefs = list(list(targets = 0, visible = TRUE)),
                                 deferRender = TRUE,
                                 scrollY = 400,
                                 scroller = TRUE,
                                 dom = 't'),
                  callback = JS("table.on('row-reorder', function(e, diff, edit) { 
                                  var arr = [];
                                  $(this).parent().find('.dataTable tbody tr').each(function() { arr.push($(this).find('td').eq(0).text()); })
                                  Shiny.onInputChange('tableReordered', arr);
                                });")
                  )
  })

  observeEvent(input$tableReordered, {
    df$data_reordered <- df$data[order(match(row.names(df$data), input$tableReordered)), ]
    df$order <- input$tableReordered
  })

  output$df_original <- renderTable(df$data, rownames = TRUE)  
  output$df_reordered <- renderTable(df$data_reordered, rownames = TRUE) 
}

ui <- fluidPage(
  column(4, 
           h4('Sortable Table'),
           DT::dataTableOutput('tab1')
         ),
  column(4, 
           h4('Original Table'),
           tableOutput('df_original')
         ),
  column(4, 
           h4('Sorted Table'),
           tableOutput('df_reordered')
         )
)

shinyApp(ui = ui, server = server)

```
{{< figure src="/images/post-images/2017-11-28-sortable_table/sortable_table.gif" >}}

