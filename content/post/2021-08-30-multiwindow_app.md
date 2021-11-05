+++
author = "Harvey"
title = "Multiwindow App"
date = "2021-08-30"
description = "Building a multiwindow shiny app"
tags = ["R", "shiny", "javascript"]
draft = true
+++

This is an initial concept of a framework for multiwindow shiny apps.  Shiny is a great R package for building an interactive, web-based, graphical UI using R.  It does, however, suffer from some limitations.  One of these is the fact that a Shiny app runs in a single browser window.  To make a more complex interface we can extend the browser window to full screen and add tab panels or scrollbars.  Another concept is to break parts of the app into separate apps, each with their own window, separate but linked through a single data source (a single source of truth).  
In this prototype, one app pushes data to a json file and two others read the data from this file using a `reactiveFileReader`.  The `reactiveFileReader` ensures that any changes made to the data updates the apps that are dependent upon it.  One additional shiny app (`launcher`) is responsible for opening and closing all other apps.

# Folder Structure

The folder structure is as follows, under the `multiwindow` project folder:

```
multiwindow
├── launcher  
│   ├── app.R
│   └── www  
│       └── script.js  
├── app_options  
│   └── app.R  
├── app_graphics  
│   └── app.R  
└── app_details  
   └── app.R  
```

There are four apps along with a javascript file which handles opening windows:
-  launcher/app.R - an app that manages opening and closing the other shiny apps as well as defining the json file location.
-  launcher/www/script.js - a javascript file that opens shiny apps in minimal browser windows of specified size and location.
-  app_options/app.R - a simple app that allows the user to select a number of options and writes them to a json file.
-  app_graphics/app.R - a simple app that reads in the json file and plots a chart.
-  app_details/app.R - a simple app that reads in the json file and outputs the file contents.

# Code

## launcher/app.R

```r
## App lauuncher
## This app manages all others


library(shiny)
library(shinyWidgets)

server <- function(input, output, session) {
  
  ## create a temp folder with access permissions
  jsonfile <- tempfile(fileext = ".json")
  
  ## replace this with the shiny server URL (subfolder = mutiwindow)
  url_base <- ###
  
  ## define app windows
  app_windows <- data.frame(
    name = c("app_options", "app_graphics", "app_details"),
    app = c("app_options", "app_graphics", "app_details"),
    height = c(0.25, 0.4, 0.25),
    width = c(0.095, 0.2, 0.095),
    left = c(0.02, 0.02, 0.125),
    top = c(0.02, 0.33, 0.02),
    closable = c(TRUE, TRUE, TRUE),
    stringsAsFactors = FALSE
  )
  app_windows$url <- paste0(url_base, app_windows$app, "/?file=", jsonfile)
  
  
  ## launch all apps
  for (i in 1:nrow(app_windows)) {
    session$sendCustomMessage("launch_app", app_windows[i, ])
  }
  
  observe({
    req(!is.null(input$txt_com_file))
    session$sendCustomMessage("disable", "txt_com_file")
  })
  
  output$ui_communications <- renderUI({
    column(10, offset = 1, textInput("txt_com_file", label = "Communications File", width = "100%", value = jsonfile))
  })
  
  
  # app_options ------------------------------------------------------------------
  
  ## options UI
  output$ui_app_options <- renderUI({
    fluidRow(
      column(1, offset = 1, style = "margin-top: 8px;", prettySwitch("swt_app_options", status = "success", label = NULL, inline = TRUE, bigger = TRUE, value = TRUE)),
      column(5, h4("shiny app: setting options")),
      column(2, offset = 1, actionBttn("but_close_all", label = "Close All", style = "simple", color = "danger", size = "sm"))
    )
  })
  
  ## switch enable
  observe({
    req(!is.null(input$swt_app_options))
    if (!app_windows[1, ]$closable) {
      session$sendCustomMessage("disable", "swt_app_options")
    }
  })
  
  ## close app_options
  observeEvent(input$swt_app_options, {
    if (input$swt_app_options == TRUE) {
      session$sendCustomMessage("launch_app", app_windows[1, ])
    } else {
      session$sendCustomMessage("close_app", app_windows[1, ])
    }
  })


  # app_graphics ------------------------------------------------------------------
  
  ## graphics UI
  output$ui_app_graphics <- renderUI({
    fluidRow(
      column(1, offset = 1, style = "margin-top: 8px;", prettySwitch("swt_app_graphics", status = "success", label = NULL, inline = TRUE, bigger = TRUE, value = TRUE)),
      column(5, h4("shiny app: plotting graphics"))
    )
  })
  
  ## switch enable
  observe({
    req(!is.null(input$swt_app_graphics))
    if (!app_windows[2, ]$closable) {
      session$sendCustomMessage("disable", "swt_app_graphics")
    }
  })
  
  ## close app_options
  observeEvent(input$swt_app_graphics, {
    if (input$swt_app_graphics == TRUE) {
      session$sendCustomMessage("launch_app", app_windows[2, ])
    } else {
      session$sendCustomMessage("close_app", app_windows[2, ])
    }
  })


  # app_details ------------------------------------------------------------------
  
  ## details UI
  output$ui_app_details <- renderUI({
    fluidRow(
      column(1, offset = 1, style = "margin-top: 8px;", prettySwitch("swt_app_details", status = "success", label = NULL, inline = TRUE, bigger = TRUE, value = TRUE)),
      column(5, h4("shiny app: setting details"))
    )
  })
  
  ## switch enable
  observe({
    req(!is.null(input$swt_app_details))
    if (!app_windows[3, ]$closable) {
      session$sendCustomMessage("disable", "swt_app_details")
    }
  })
  
  ## close app_options
  observeEvent(input$swt_app_details, {
    if (input$swt_app_details == TRUE) {
      session$sendCustomMessage("launch_app", app_windows[3, ])
    } else {
      session$sendCustomMessage("close_app", app_windows[3, ])
    }
  })
  
  ## close all apps
  observeEvent(input$but_close_all, {
    for (i in 1:nrow(app_windows)) {
      session$sendCustomMessage("close_app", app_windows[i, ])
    }
  })

}

ui <- fluidPage(
  tags$head(
    tags$script(type = "text/javascript", src = "script.js")
  ),
  br(),
  br(),
  fluidRow(column(10, offset = 1, 
                  panel(status = "primary", heading = "App Launcher",
                        panel(status = "danger", heading = "Communications",
                              uiOutput("ui_communications")
                        ),
                        br(),
                        panel(status = "danger", heading = "App Windows",
                              fluidRow(uiOutput("ui_app_options")),
                              fluidRow(uiOutput("ui_app_graphics")),
                              fluidRow(uiOutput("ui_app_details"))
                        )
                  )
  ))
)

shinyApp(ui = ui, server = server)

```

## launcher/www/script.js

```js
var shiny_app_options = "";
var shiny_app_graphics = "";
var shiny_app_details = "";

// launch a shiny app in a minimal window
// window opens with a specified size at a specified screen location (based on fraction of total screen width and height)
Shiny.addCustomMessageHandler('launch_app', function(x) {
  scr_height = window.screen.height;
  scr_width = window.screen.width;
  window_height = scr_height * x.height;
  window_width = scr_width * x.width;
  window_left = scr_width * x.left;
  window_top = scr_height * x.top;
  window_options = "height=" + window_height + ", width=" + window_width + ", left=" + window_left + ", top=" + window_top;
  
  if (x.name == "app_options") {
    shiny_app_options = window.open(x.url, x.name, window_options);
  } else if (x.name == "app_graphics") {
    shiny_app_graphics = window.open(x.url, x.name, window_options);
  } else if (x.name == "app_details") {
    shiny_app_details = window.open(x.url, x.name, window_options);
  }
});

// close a shiny app
Shiny.addCustomMessageHandler('close_app', function(x) {
  console.log(x.name);
  // can't pass window name as variable to close so have to hardcode :(
  if (x.name == "app_options") {
    console.log('close app_options');
    shiny_app_options.close();
  } else if (x.name == "app_graphics") {
    console.log('close app_graphics');
    shiny_app_graphics.close();
  } else if (x.name == "app_details") {
    console.log('close app_details');
    shiny_app_details.close();
  }
});

// disable a shiny input
Shiny.addCustomMessageHandler('disable', function(id) {
  var input_type = $("#" + id).prop("type");
  if (input_type.startsWith("select")) {
    $("#" + id)[0].selectize.disable();
  } else {
    $("#" + id).prop("disabled", true);
  }
});

```

