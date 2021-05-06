+++
author = "Harvey"
title = "Alerts in Chrome and IE"
date = "2019-02-19"
description = "Displaying Browser-Dependent alerts in Shiny Apps"
tags = ["R", "shiny", "javascript"]
+++

The sendSweetAlert function from the shinyWidgets package is a great way to communicate behavior to an end-user.  As of version 0.4.9 shinyWidgets now uses SweetAlert2 which is no longer compatible with Internet Explorer thus breaking historical apps which use this feature.  This is due to the fact that IE does not support arrow functions.  
One workaround is to switch from using sweetalert to using shiny's built in showNotification function.  This function works under both Chrome and IE but is not as customizable or beautiful as sweetalert.  Another alternative is to identify the browser and provide a sweetalert if running Chrome or shiny notification if not.  This is achieved by using javascript to assign a shiny **INPUT** (input$chrome) variable and using this value to determine whch type of notification to show.


### Included in  UI
```r
tags$script(HTML(
      '$(document).on("shiny:sessioninitialized", function(event) {
      var runningChrome = /Chrome/.test(navigator.userAgent);
      Shiny.setInputValue("chrome", runningChrome);
    });'
```

### Included in server
```r
show_alert <- function(session = getDefaultReactiveDomain(), chrome_browser = FALSE, title = NULL, text = NULL, type = 'error', btn_labels = NULL) {
  if (!type %in% c('info', 'success', 'warning', 'error')) type <- 'info'
  if (chrome_browser) {
    if (!is.null(btn_labels)) {
      sendSweetAlert(session = session, title = title, text = text, type = type, btn_labels = btn_labels, closeOnClickOutside = TRUE)
    } else {
      sendSweetAlert(session = session, title = title, text = text, type = type, closeOnClickOutside = TRUE)
    }
  } else {
    text <- str_replace(text, '\\n', '<br>')
    if (type == 'info') type <- 'default'
    if (type == 'success') type <- 'message'
    showNotification(session = session, ui = title, action = HTML(text), type = type)
  }
}
```
The show\_alert function takes a session variable, chrome\_browser logical and some additional pararmeters to build a notification.

