+++
author = "Harvey"
title = "Mimic Button Press with Enter Key"
date = "2018-11-09"
description = "Minic a button press with the Enter key"
tags = ["R", "shiny", "javascript"]
+++

Sometimes it's useful to mimic a button press in a shiny app when the Enter key is pressed, for example when entering a password.  This has been [documented before](https://github.com/daattali/advanced-shiny/blob/master/proxy-click/app.R) and works well for most situations but fails when the button widget is housed within renderUI.  Here's an alternative that works wherever the password input and button are located.

```r
library(shiny)
library(shinyjs)
library(shinyWidgets)

jscode <- '
  shinyjs.setbutton = function(params) {
    var defaultParams = {
      id: null,
      button: null
    };
    params = shinyjs.getParams(params, defaultParams);
    var el = $("#" + params.id);
    var button = $("#" + params.button);
    el.keyup(function(event) {
      if (event.keyCode === 13) {
        button.click();
      }
    })
  };'

ui <- fluidPage(
  useShinyjs(),
  extendShinyjs(text = jscode, functions = c('setbutton')),
  tags$head(tags$script(HTML(jscode))),
  fluidRow(
    br(),
    column(4, offset = 4, uiOutput('loginpanel')),
    br(),
    verbatimTextOutput('txtout')
  )
)


server <- function(input, output, session) {
  
  ## reactiveValue to hold username and password
  out <- reactiveValues(user = NULL, pw = NULL)
  
  output$loginpanel <- renderUI({
    panel(heading = 'login', status = 'danger',
          selectInput('user', label = NULL, choices = c('bob', 'ben', 'bill')),
          textInput('txt1', label = 'textbox', value =''),  ## textbox to show that hitting Enter here has no effect
          uiOutput('pw'),
          actionBttn('butLogin', label = 'login', style = 'simple', color = 'success'))
  })
  
  output$pw <- renderUI({
    passwordInput('pwinp', label = NULL)
  }) 
  
  ## attach an event to the passwordInput after a delay to ensure that the widget has been rendered
  delay(100, 
        js$setbutton('pw', 'butLogin')
  )

  ## triggered when button pressed or Enter key when focussed on passwordInput widget
  observeEvent(input$butLogin, {
    out$user <- input$user
    out$pw <- input$pwinp
  })
  
  output$txtout <- renderPrint({
    paste(out$user, out$pw)
  })
}

shinyApp(ui = ui, server = server)
```