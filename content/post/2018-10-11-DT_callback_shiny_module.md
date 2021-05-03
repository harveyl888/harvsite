+++
author = "Harvey"
title = "Datatable Callback in Shiny Module"
date = "2018-10-11"
description = "Datatable Callback in Shiny Module"
tags = ["R", "shiny", "shiny modules"]
+++

Using shiny modules a large shiny application can be broken down into manageable chunks.  Modules work well with most aspects of shiny code but care needs to be taken when referencing inputs.  This is especially true when javascript is used within a module.  Here's a piece of code which uses a callback in a datatable to capture the current and previous row (something I use to allow automatic updating of a database when a row is changed).  The important point is to remember to use *session$ns* in the server code whenever referencing an input created in the module.

### Code outside of a modules
```r
## callback to capture current and previous row - allows automatic updating of database when a new row is selected
callback = JS(
  "table.on('click.dt', 'tr', function() {
    $(this).toggleClass('active');
    if (typeof(current_row) === 'undefined') {
      Shiny.onInputChange('assay_previous_row', null);
    } else {
      Shiny.onInputChange('assay_previous_row', current_row);
    }
    current_row = table.rows('.selected').data()[0][0];
    Shiny.onInputChange('assay_selected_row', current_row);
  });"
)
```

### Code within a shiny modules
```r
## callback to capture current and previous row - allows automatic updating of database when a new row is selected
callback = JS(paste0(
  "table.on('click.dt', 'tr', function() {
    $(this).toggleClass('active');
    if (typeof(current_row) === 'undefined') {
      Shiny.onInputChange('", session$ns('assay_previous_row'), "', null);
    } else {
      Shiny.onInputChange('", session$ns('assay_previous_row'), "', current_row);
    }
    current_row = table.rows('.selected').data()[0][0];
    Shiny.onInputChange('", session$ns('assay_selected_row'), "', current_row);
  });"
))
```

This aproach can also be applied for functions within tables.  For example here's code for a delete button within a table.

### Code outside of a modules
```r
delete = as.character(actionButton(paste0('newassaydelete_', newassay$counter), label = NULL, icon = icon('trash'), onclick = paste0('Shiny.onInputChange(\"delete_new_assay\", this.id)'), style="color: #fff; background-color: #cc0000; border-color: #660000"))
```
### Code within a shiny modules
```r
delete = as.character(actionButton(session$ns(paste0('newassaydelete_', newassay$counter)), label = NULL, icon = icon('trash'), onclick = paste0('Shiny.onInputChange(\"', session$ns('delete_new_assay'), '\", this.id)'), style="color: #fff; background-color: #cc0000; border-color: #660000"))
```