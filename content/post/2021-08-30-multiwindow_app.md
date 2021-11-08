+++
author = "Harvey"
title = "Multiwindow App"
date = "2021-08-30"
description = "Building a multiwindow shiny app"
tags = ["R", "shiny", "javascript"]
codeMaxLines = 15
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

# Concept

launcher controls opening and closing of the other three apps (app_options, app_graphics and app_details).  Communication is handled using a json file.

{{< mermaid align="left" theme="neutral" >}}
graph TB
  subgraph launcher
    app_options --> json_file;
    json_file --> app_graphics;
    json_file --> app_details;
  end
{{< /mermaid >}}

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


## app_options/app.R

```r
## app_options
## A simple app that offers a series of options

library(shiny)
library(shinyWidgets)
library(jsonlite)

server <- function(input, output, session) {
  
  filename <- reactive({
    query <- parseQueryString(session$clientData$url_search)
    if (!is.null(query$file)) {
      query$file
    } else {
      ""
    }
  })
  
  output$ui_axes <- renderUI({
    req(input$sel_data)
    ## select x and y parameters from numeric columns
    df <- get(input$sel_data)
    columns <- names(df)[sapply(df, class) == "numeric"]
    tagList(
      selectizeInput("sel_x", "X parameter", choices = columns),
      selectizeInput("sel_y", "Y parameter", choices = columns)
    )
  })
  
  ## write file when updating parameter
  observeEvent(input$sel_data, {
    write_file()
  })
  
  observeEvent(input$sel_x, {
    write_file()
  })
  
  observeEvent(input$sel_y, {
    write_file()
  })
  
  ## write a file
  write_file <- function() {
    if (filename() != "") {
      output <- list(
        data = input$sel_data,
        x = input$sel_x,
        y = input$sel_y
      )
      json_out <- toJSON(output, auto_unbox = TRUE, null = "null")
      con <- file(filename(), open = "wt")
      writeLines(json_out, con)
      close(con)
    }
  }
}

ui <- fluidPage(
  br(),
  br(),
  panel(heading = "Options", status = "primary",
        selectizeInput("sel_data", "dataset", choices = c("iris", "mtcars")),
        uiOutput("ui_axes")
        )
)

shinyApp(ui, server)
```

## app_graphics/app.R

```r
## app_graphics
## A simple app that draws a ggplot

library(shiny)
library(shinyWidgets)
library(ggplot2)
library(jsonlite)

server <- function(input, output, session) {
  
  filename <- reactive({
    query <- parseQueryString(session$clientData$url_search)
    if (!is.null(query$file)) {
      query$file
    } else {
      ""
    }
  })
  
  ## retrieve data from file
  json_data <- reactiveFileReader(2000, session, filePath = filename, readFunc = function(filePath) {
    tryCatch({
      con <- file(filePath)
      file_contents <- readLines(con)
      close(con)
      file_contents
    },
    error = function(e) NA
    )
  })
  
  output$plt <- renderPlot({
    req(json_data())
    
    ## check all plotting parameters are present
    params <- c("data", "x", "y")
    data <- fromJSON(json_data())
    if (all(sapply(params, function(x) !is.null(data[[x]])))) {
      ggplot(get(data[["data"]]), aes(x = .data[[data[["x"]]]], y = .data[[data[["y"]]]])) +
        geom_point() +
        labs(title = paste0("Plot data = ", data[["data"]]),
             x = data[["x"]], 
             y = data[["y"]])
    }

  })
  
}

ui <- fluidPage(
  br(),
  br(),
  panel(heading = "Graphics", status = "primary",
        plotOutput("plt")
  )
)

shinyApp(ui, server)
```

## app_details/app.R

```r
## app_details
## A simple app that lists some details

library(shiny)
library(shinyWidgets)
library(jsonlite)

server <- function(input, output, session) {
  
  filename <- reactive({
    query <- parseQueryString(session$clientData$url_search)
    if (!is.null(query$file)) {
      query$file
    } else {
      ""
    }
  })
  
  ## retrieve data from file
  json_data <- reactiveFileReader(2000, session, filePath = filename, readFunc = function(filePath) {
    tryCatch({
      con <- file(filePath)
      file_contents <- readLines(con)
      close(con)
      file_contents
    },
    error = function(e) NA
    )
  })
  
  output$txt_details <- renderPrint({
    prettify(json_data())
  })
  
}

ui <- fluidPage(
  br(),
  br(),
  panel(heading = "Output", status = "primary",
        verbatimTextOutput("txt_details")
  )
)

shinyApp(ui, server)
```

# Output

When running the `launcher` app the following window first opens:

{{< figure src="/images/post-images/2021-08-30-multiwindow_app/multiwindow_01.png" >}}

This automatically triggers opening of the other three shiny apps (app_options, app_graphics and app_details).  The windows for these apps are sized and located according to the `app_windows` data frame in launcher/app.R as follows:

```r
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
```

Here, the sizes and dimensions are based on screen fractions so app_options is 25% of the screen height and 9.5% of the screen width, and is placed 2% from the left of the screen and 2% from the top of the screen.

The resulting screen capture of app_options, app_graphics and app_details looks like this:

{{< figure src="/images/post-images/2021-08-30-multiwindow_app/multiwindow_02.png" >}}

When an option in app_options is changed, the json file is update and the `reactiveFileReader` in app_graphics and app_details triggers an update.  All three apps are independent but connected through the json file.

Here's a screenshot after updating:

{{< figure src="/images/post-images/2021-08-30-multiwindow_app/multiwindow_03.png" >}}

