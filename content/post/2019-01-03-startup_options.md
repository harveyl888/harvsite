+++
author = "Harvey"
title = "Shiny app startup options"
date = "2019-01-03"
description = "Startup a shiny app with options in interactive or server mode"
tags = ["R", "shiny"]
+++

Using `parseQueryString` in conjunction with `session$clientData$url_search` can be used to pass variables to a shiny app from the url address.  It can also be used with `Sys.getenv('SHINY_PORT')` to follow programming paths based on variable values and execution environment.  In the example below, the code points to different instances of a database, depending on user-specification and execution environment.  Running locally or on the server points to different default databases and includes the ability to override.  This code is useful when placed at the top of an app during development.

```r
library(shiny)

## allowed_databases stores the names of the allowable databases
allowed_databases <- c('test', 'production', 'uploadtest')

## default database names for interactive and server modes
dafault_interactive <- 'test'
dafault_server <- 'production'

## override database defaults for interactive mode
## to override the default database set override_default_database to TRUE and 
## override_use_database to the database name
override_default_database <- FALSE
override_use_database <- 'production'

server <- function(input, output, session) {
  
  val <- reactiveValues(db = NULL)
  
  db_set_database <- function(name) {
    val$db <- name
  }
  
  ## set database
  observe({
    if(Sys.getenv('SHINY_PORT') == "") {
      ## if running locally then use test database unless override set
      if (override_default_database) {
        ## run against specified instance
        if (!override_use_database %in% allowed_databases) override_use_database <- dafault_interactive
        db_set_database(override_use_database)
      } else {
        ## run against default interactive instance
        db_set_database(dafault_interactive)
      }
    } else {
      ## if running on server then look for database in url
      query <- parseQueryString(session$clientData$url_search)
      if (!is.null(query[['database']])) {
        ## get database name from url and check that it is in the list of allowed database names
        my_database <- tolower(query[['database']])
        if (!my_database %in% allowed_databases) my_database <- dafault_server
        ## run against selected instance
        db_set_database(my_database)
      } else {
        ## default - run against defaukt server instance
        db_set_database(dafault_server)
      }
    }
  })
  
  output$txt1 <- renderPrint(paste0('Using database ', val$db))
  
}

ui <- fluidPage(
  verbatimTextOutput('txt1')
)

shinyApp(ui, server)
```