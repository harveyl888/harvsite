+++
author = "Harvey"
title = "Shiny Background Process"
date = "2017-04-08"
description = "Running a background process under Shiny"
tags = ["R", "shiny"]
+++

R is a very powerful language but was designed to run single-threaded.  There are libraries that will run parallel code but there is no inherent support to run a background process and monitor for completion in the same way that you can in, for example, C++.  

One way to overcome this is to launch a second instance of R and run the process in the background.  If the background process generates a log file it can be monitored within Shiny using the reactivePoll or reactiveFileInput functions.  Once completed subsequent action can be taken.  The advantage of this method is that the second process is run under a second instance of R and does not interfere with the user interface of the Shiny app.  

By way of example, there are two apps below.  The first attempts to run the process within an observer which ties up the UI – the UI will only respond to changes once the calculation is completed.  The second launches a second instance and the UI is not affected – the UI is fully responsive during the calculation.  


## Running as a Single Process
The gist for the code can be found at [https://gist.github.com/harveyl888/fa6ff9823b9c5a5fff11c946d8e7c9f5](https://gist.github.com/harveyl888/fa6ff9823b9c5a5fff11c946d8e7c9f5)

```r
## Data creation
## 
## Create a large Excel spreadsheet within a Shiny app
##

library(shiny)
library(openxlsx)

## Create a dummy matrix

server <- function(input, output, session) {
  
  mydata <- reactiveValues(wb = NULL)
  status <- reactiveValues(text = 'Waiting')
  
  ## Disable download button
  observe({
    session$sendCustomMessage('disableButton', 'butDownload')
  })

  ## Generate Excel output
  observeEvent(input$butCreate, {
    session$sendCustomMessage('disableButton', 'butDownload')
    session$sendCustomMessage('disableButton', 'butCreate')
    
    ## Included for comparison - the status text will not update until after the spreadsheet is built
    status$text <- 'Building'
    
    m <- matrix(rexp(input$numRows * input$numCols, rate = 0.1), ncol = input$numCols)
    
    wb <- createWorkbook()
    addWorksheet(wb, 'sheet1')
    writeData(wb, 'sheet1', m)
    mydata$wb <<- wb
    session$sendCustomMessage('enableButton', 'butDownload')
    session$sendCustomMessage('enableButton', 'butCreate')
    status$text <- 'Completed'
  })
  
  output$butDownload <- downloadHandler(
    filename = function() {
      'output.xlsx'
    },
    content = function(file) {
      showNotification('Writing Excel File')
      saveWorkbook(mydata$wb, file, overwrite = TRUE)
    }
  )
  
  output$uiStatus <- renderUI(
    h4(paste0('STATUS: ', status$text), style="color:red;")
  )

  output$plt <- renderPlot({
    hist(faithful$eruptions, probability = TRUE, breaks = as.numeric(input$n_breaks),
         xlab = 'Duration (minutes)', main = 'Geyser eruption duration')
    dens <- density(faithful$eruptions, adjust = input$bw_adjust)
    lines(dens, col = 'blue')
  })
}

ui <- fluidPage(
  singleton(tags$head(HTML('
    <script type="text/javascript">
    $(document).ready(function() {
      // Enable button
      Shiny.addCustomMessageHandler("enableButton", function(id) {
        $("#" + id).removeAttr("disabled");
      });
      // Disable button
      Shiny.addCustomMessageHandler("disableButton", function(id) {
        $("#" + id).attr("disabled", "true");
      });
      })
    </script>
    ')
  )),
  fluidRow(
    column(4,
           wellPanel(
             fluidRow(
               column(6, numericInput('numRows', 'Number of Rows', value = 100000, min = 1000, step = 1000)),
               column(6, numericInput('numCols', 'Number of Columns', value = 50, min = 10, step = 10))
             )
           ),
           fluidRow(
             column(11, offset = 1,
                    actionButton('butCreate', 'Create Data', class = 'btn action-button btn-success'),
                    downloadButton('butDownload', 'Download Data', class = 'btn btn-warning'),
                    br(),
                    uiOutput('uiStatus')
             )
           )
    ),
    column(8,
           wellPanel(
             fluidRow(
               column(4, selectInput('n_breaks', label = 'Number of bins:', choices = c(10, 20, 35, 50), selected = 20)),
               column(4, sliderInput('bw_adjust', label = 'Bandwidth adjustment:', min = 0.2, max = 2, value = 1, step = 0.2))
             )
           ),
           plotOutput('plt')
           )
    )
  )

shinyApp(ui = ui, server = server)
```

## Running as an Asynchronous Process
The gist for the code can be found at [https://gist.github.com/harveyl888/bf05d902b10c138a02acd5c9c65fc5da](https://gist.github.com/harveyl888/bf05d902b10c138a02acd5c9c65fc5da)

```r
## Data creation
## 
## Create a large Excel spreadsheet as a asynchronous process
##

library(shiny)

## Temporary files to store log, script, rds data and excel output
logfile <- tempfile()
scriptfile <- tempfile()
datafile <- tempfile()
excelfile <- tempfile()

server <- function(input, output, session) {

  status <- reactiveValues(text = 'Waiting')
    
  ## Disable download button
  observe({
    session$sendCustomMessage('disableButton', 'butDownload')
  })
  
  # reactivePoll - look for changes in log file every second
  logData <- reactivePoll(1000, session,
                          checkFunc = function() {
                            if (file.exists(logfile))
                              file.info(logfile)$mtime[1]
                            else
                              ''
                            },
                          valueFunc = function() {
                            if (file.exists(logfile))
                              readLines(logfile)
                            else
                              ''
                            }
  )

  ## React to an update in the logfile
  observe({
    if (grepl('COMPLETED', logData())) {
      session$sendCustomMessage('enableButton', 'butDownload')
      session$sendCustomMessage('enableButton', 'butCreate')
      status$text <- 'Completed'
    } 
  })

  ## Generate Excel output
  ## Once button is pressed create an R Script and run as a second process
  ## to avoid tying up Shiny
  observeEvent(input$butCreate, {
    session$sendCustomMessage('disableButton', 'butDownload')
    session$sendCustomMessage('disableButton', 'butCreate')
    status$text <- 'Building'

    m <- matrix(rexp(input$numRows * input$numCols, rate = 0.1), ncol = input$numCols)
    
    ## Write data to an rds file
    saveRDS(m, file = datafile)
    
    ## Create script file
    vfile <- c('library(openxlsx)',
               paste0('m <- readRDS(\"', datafile, '\")'),
               'wb <- createWorkbook()',
               'addWorksheet(wb, \"sheet1\")',
               'writeData(wb, \"sheet1\", m)',
               paste0('saveWorkbook(wb, \"', excelfile, '\", overwrite = TRUE)'),
               paste0('fileConn <- file(\"', logfile, '\")'),
               'writeLines(\"COMPLETED\", fileConn)',
               'close(fileConn)'
               )
    
    ## Save script file
    fileConn <- file(scriptfile)
    writeLines(vfile, fileConn)
    close(fileConn)
    
    ## Execute script file
    system(paste0(Sys.getenv('R_HOME'), '/bin/Rscript ', scriptfile), wait = FALSE)
  })
  
  output$butDownload <- downloadHandler(
    filename <- function() {
      'excel-out.xlsx'
    },
    content <- function(file) {
      file.copy(excelfile, file)
    }
  )
  
  output$uiStatus <- renderUI(
    h4(paste0('STATUS: ', status$text), style="color:red;")
  )
  
  output$plt <- renderPlot({
    hist(faithful$eruptions, probability = TRUE, breaks = as.numeric(input$n_breaks),
         xlab = 'Duration (minutes)', main = 'Geyser eruption duration')
    dens <- density(faithful$eruptions, adjust = input$bw_adjust)
    lines(dens, col = 'blue')
  })
}

ui <- fluidPage(
  singleton(tags$head(HTML('
    <script type="text/javascript">
    $(document).ready(function() {
      // Enable button
        Shiny.addCustomMessageHandler("enableButton", function(id) {
      $("#" + id).removeAttr("disabled");
      });
      // Disable button
        Shiny.addCustomMessageHandler("disableButton", function(id) {
      $("#" + id).attr("disabled", "true");
      });
    })
    </script>
    ')
  )),
  fluidRow(
    column(4,
           wellPanel(
             fluidRow(
               column(6, numericInput('numRows', 'Number of Rows', value = 100000, min = 1000, step = 1000)),
               column(6, numericInput('numCols', 'Number of Columns', value = 50, min = 10, step = 10))
             )
           ),
           fluidRow(
             column(11, offset = 1,
               actionButton('butCreate', 'Create Data', class = 'btn action-button btn-success'),
               downloadButton('butDownload', 'Download Data', class = 'btn btn-warning'),
               br(),
               uiOutput('uiStatus')
             )
           )
    ),
    column(8,
           wellPanel(
             fluidRow(
               column(4, selectInput('n_breaks', label = 'Number of bins:', choices = c(10, 20, 35, 50), selected = 20)),
               column(4, sliderInput('bw_adjust', label = 'Bandwidth adjustment:', min = 0.2, max = 2, value = 1, step = 0.2))
             )
           ),
           plotOutput('plt')
           )
    )
  )

shinyApp(ui = ui, server = server)
```
