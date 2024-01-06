+++
author = "Harvey"
title = "Adding to a sortable bucketlist"
date = "2023-11-10"
description = "Dynamically updating a sortable bucketlist upon adding a new element"
tags = ["R", "shiny"]
codeMaxLines = 15
draft = false
+++

# The {sortable} package

[{sortable}](https://rstudio.github.io/sortable/) is an incredibly useful R package built upon the [sortbale.js](https://sortablejs.github.io/Sortable/) javascript library, allowing drag-and-drop features to be incorporated into shiny apps.  The package works with two types of sortable lists:

-  rank list: items can be sorted within a list
-  bucket list: a single object containing multiple rank lists, allowing for drag-and-drop between lists.

The second option (bucket list) is useful when a user wishes to divide a set of values between two or more buckets.  {sortable} integrates well within shiny but requires a few tweaks if data are added to the sortable object once it has been created.

{{% notice note "Note" %}}
{sortable} includes a function, `update_bucket_list()` which is used to update header text but not list items.
{{% /notice %}}

# Case One.  Dragging Existing Values Between Lists

Here's a simple scenario.  Suppose you have a list of objects in *list 1* and you wish to subset to *list 2*, returning the values in *list 2*.  Using {sortable} you could define a bucket list containing two rank lists (*list 1* and *list 2*), populate *list 1* and drag-and-drop values to *list 2*.  In a shiny app this could be coded as follows:

```r
library(shiny)
library(sortable)

ui <- fluidPage(
  uiOutput("ui_sort"),
  verbatimTextOutput("txt_output")
)

server <- function(input, output, session) {
  
  output$ui_sort <- renderUI({
    
    ## create styled list 1
    list_1_tags <- lapply(LETTERS[1:5], function(x) {
      tags$span(id = paste0('lab_', x), x, class = "label label-primary", `data-rank-id` = x)
    })
    
    ## create styled list 2
    list_2_tags <- lapply(LETTERS[6:8], function(x) {
      tags$span(id = paste0('lab_', x), x, class = "label label-primary", `data-rank-id` = x)
    })
    
    ## sortbale bucket list
    sortable::bucket_list(
      header = "Move values from list 1 to list 2",
      group_name = "reorder_list",
      sortable::add_rank_list(
        input_id = "reorder_list_1",
        text = "list 1",
        labels = list_1_tags
      ),
      sortable::add_rank_list(
        input_id = "reorder_list_2",
        text = "list 2",
        labels = list_2_tags
      ))
  })
  
  output$txt_output <- renderPrint({
    print(input$reorder_list)
  })
  
}

shinyApp(ui, server)
```

In this example, every time *list 1* or *list 2* changes, *input$reorder_list* updates with the ordered values for each list.

# Case Two.  Adding a New Value to an Existing List

Updating the code above with an **actionButton** linked to **insertUI** allows us to add a new option to list 2 when the button is pressed.  This approach works but you'll see that it does not update *input$reorder_list* until the list is updated (by moving an item).

```r
library(shiny)
library(sortable)

ui <- fluidPage(
  uiOutput("ui_sort"),
  actionButton("but_add", "Add"),
  verbatimTextOutput("txt_output")
)

server <- function(input, output, session) {
  
  output$ui_sort <- renderUI({
    
    ## create styled list 1
    list_1_tags <- lapply(LETTERS[1:5], function(x) {
      tags$span(id = paste0('lab_', x), x, class = "label label-primary", `data-rank-id` = x)
    })
    
    ## create styled list 2
    list_2_tags <- lapply(LETTERS[6:8], function(x) {
      tags$span(id = paste0('lab_', x), x, class = "label label-primary", `data-rank-id` = x)
    })
    
    ## sortbale bucket list
    sortable::bucket_list(
      header = "Move values from list 1 to list 2",
      group_name = "reorder_list",
      sortable::add_rank_list(
        input_id = "reorder_list_1",
        text = "list 1",
        labels = list_1_tags
      ),
      sortable::add_rank_list(
        input_id = "reorder_list_2",
        css_id = "css_reorder_list_2",
        text = "list 2",
        labels = list_2_tags
      ))
  })
  
  observeEvent(input$but_add, {
    ## insert a new value into list 2
    val <- "ZZ"
    new_tag <- tags$div(class = "rank-list-item", draggable = FALSE,
                        tags$span(id = paste0('lab_', val), val, class = "label label-primary", `data-rank-id` = val)
    )
    insertUI(selector = "#css_reorder_list_2", where = "beforeEnd", ui = new_tag, immediate = TRUE)
  })
  
  output$txt_output <- renderPrint({
    print(input$reorder_list)
  })
  
}

shinyApp(ui, server)
```

{{< figure src="/images/post-images/2023-11-10-sortable_add/sortable_01.gif" >}}

In order to make this approach work we can manage the ordered list, *list 2* through a shiny input, *input$sort_list_2*.  This shiny variable is kept up to date through two methods: 

-  `sortable::sortable_options()` updates *input$sort_list_2* when *list 2* is first initialized, a new item is dragged from *list 1* or the order is changed.
-  When the new item is added, a javascript function is executed, updating *input$sort_list_2* with the newly ordered list.  The javascript function works but identifying the identifer of the list and looping through its members, populating *input$sort_list_2*.


```r
library(shiny)
library(sortable)

ui <- fluidPage(
  
  tagList(
    tags$head(tags$script(src = "script.js")),
    uiOutput("ui_sort"),
    actionButton("but_add", "Add"),
    verbatimTextOutput("txt_output")
  )
  
)

l1 <- LETTERS[1:5]
l2 <- LETTERS[6:8]

server <- function(input, output, session) {
  
  output$ui_sort <- renderUI({
    
    list_1_tags <- lapply(l1, function(x) {
      tags$span(id = paste0('lab_', x), x, class = "label label-primary", `data-rank-id` = x)
    })
    
    list_2_tags <- lapply(l2, function(x) {
      tags$span(id = paste0('lab_', x), x, class = "label label-primary", `data-rank-id` = x)
    })
    
    sortable::bucket_list(
      header = "Reorder values in 'ordered values' column",
      group_name = "reorder_list",
      sortable::add_rank_list(
        input_id = "reorder_list_1",
        text = "list 1",
        labels = list_1_tags
      ),
      sortable::add_rank_list(
        input_id = "reorder_list_2",
        css_id = "css_reorder_list_2",
        text = "list 2",
        labels = list_2_tags,
        options = sortable_options(
          onSort = sortable_js_capture_input("sort_list_2"),
          onLoad = sortable_js_capture_input("sort_list_2"))
      ))
  })
  
  observeEvent(input$but_add, {
    print("adding ZZ")
    val <- "ZZ"
    new_tag <- tags$div(class = "rank-list-item", draggable = FALSE,
                        tags$span(id = paste0('lab_', val), val, class = "label label-primary", `data-rank-id` = val)
    )
    insertUI(selector = "#css_reorder_list_2", where = "beforeEnd", ui = new_tag, immediate = TRUE)
    session$sendCustomMessage('update_sortable', list(id = "css_reorder_list_2", shinyinput = "sort_list_2"))
  })
  
  output$txt_output <- renderPrint({
    print(input$sort_list_2)
  })
  
}

shinyApp(ui, server)
```

```javascript
Shiny.addCustomMessageHandler('update_sortable', function(x) {
  if (typeof Shiny !== 'undefined') {
    el = document.getElementById(x.id);
    shinyinputname = x.shinyinput + ':sortablejs.rank_list'
    Shiny.setInputValue(shinyinputname, $.map(el.children, function(child) {
      return $(child).attr('data-rank-id') || $.trim(child.innerText);
    }))
  }
})
```

{{< figure src="/images/post-images/2023-11-10-sortable_add/sortable_02.gif" >}}

# Conclusion
The code above demonstrates a suitable approach to add items to a sortable bucket list in a shiny app.  The concept can be extended to work with multiple lists, returning the content of each.

