+++
author = "Harvey"
title = "Managing Users"
date = "2017-01-13"
description = "Managing users in a Shiny Ap"
tags = ["R", "shiny"]
+++

There are several ways that users can be managed in Shiny apps but they all require access to the config file.  User access can be managed within the app itself using a database to hold user information.  Two examples are given below:  The first utilizes a simple database in which the passwords are not encrypted (not recommended for multiple reasons!)  The second is very similar but it uses the sodium library to encode the passwords in the database.  These are very simplistic examples and contain no functions for password management (such as users being able to change their passwords or password reset) but these would be straight forward to add.

## User login - No encrpytion
[https://gist.github.com/harveyl888/a85e17c3048e0da03cf4e6b52d1da3db](https://gist.github.com/harveyl888/a85e17c3048e0da03cf4e6b52d1da3db)

```r
## Authentication
## This is a small app to demonstrate user-managed authentication without encoded passwords.
## Users are stored in a SQL database with passwords along with roles.  
## Once a user is logged in the shiny app responds to the user's role.
## In order to use in a real setting, additional code for password management,
## changing and resetting would need to be implemented.

library(shiny)
library(RSQLite)

## create the initial password database
## This code should be run once to create the initial database of users, passwords and roles
##
# db.pw <- data.frame(user = c('Augustin', 'Matt', 'Harvey'), role = c('Manager', 'User', 'User'), password = c('ABC', 'DEF', 'GHI'))
# db = dbConnect(SQLite(), dbname = 'auth_nohash.sqlite')
# dbSendQuery(db, 'CREATE TABLE pw (user TEXT, password TEXT, role TEXT)')
# apply(db.pw, 1, function(x) dbSendQuery(db, paste0('INSERT INTO pw VALUES("', x['user'], '", "', x['password'], '", "', x['role'], '")')))
# dbDisconnect(db)

## Connect to the database (may be a remote connection)
db = dbConnect(SQLite(), dbname = 'auth_nohash.sqlite')

server <- function(input, output, session) {
  
  ## Initialize - user is not logged in
  user <- reactiveValues(login = FALSE, name = NULL, role = NULL, header = NULL)
  
  ## Display login modal
  observe({
    showModal(modalDialog(
      title = "Enter Login Details",
      textInput('userInp', 'Login'),
      passwordInput('pwInp', 'Password'),
      actionButton('butLogin', 'Login', class = 'btn action-button btn-success', icon = icon('sign-in')),
      size = 's',
      easyClose = FALSE,
      footer = NULL
    ))
  })
  
  ## Check for user in database
  observeEvent(input$butLogin, {  ## login button pressed
    req(input$userInp, input$pwInp)  ## ensure we have inputs
    removeModal()  ## remove the modal
    pw_out <- dbGetQuery(db, paste0('SELECT password FROM pw WHERE user = \"', input$userInp, '\"'))  ## query database
    if (nrow(pw_out) == 0) {  ## user does not exist
      user$login <- FALSE
      user$header <- 'ERROR - UNKNOWN USER'
    } else {
      pw <- as.character(pw_out$password)[[1]]  ## grab password from database
      passwordVerified <- pw == input$pwInp  ## check that it matches user input
      if (passwordVerified) {  ## match
        user$login <- TRUE
        user$name <- input$userInp
        user$role <- db.pw[db.pw$user == input$userInp, 'role']
        user$header <- paste0(user$name, ' (', user$role, ')')
      } else {  ## no match
        user$login <- FALSE
        user$header <- 'ERROR - INCORRECT PASSWORD'
      }
    }
  })
  
  ## close database on exit
  session$onSessionEnded(function(){
    dbDisconnect(db)
  })
  
  output$data <- renderUI({
    h4(user$header)
  })
  
  output$myPlot <- renderPlot({
    req(user$login)
    if (user$role == 'Manager') {  ## If manager role, display iris plot
      plot(iris$Sepal.Length, iris$Sepal.Width)
    } else {  ## If user role, display mtcars plot
      plot(mtcars$mpg, mtcars$cyl)
    }
  })
  
}

ui <- fluidPage(
  uiOutput('data'),
  plotOutput('myPlot')
)

shinyApp(ui = ui, server = server)
```

## User login - With encrpytion
[https://gist.github.com/harveyl888/3e5123a6469fbdc3830123e3efb31a2a](https://gist.github.com/harveyl888/3e5123a6469fbdc3830123e3efb31a2a)

```r
## Authentication
## This is a small app to demonstrate user-managed authentication using a hash to encode passwords.
## Users are stored in a SQL database with passwords along with roles.
## Once a user is logged in the shiny app responds to the user's role.
## In order to use in a real setting, additional code for password management,
## changing and resetting would need to be implemented.

library(shiny)
library(RSQLite)
library(sodium)

## create the initial password database
## This code should be run once to create the initial database of users, passwords and roles
##
# db.pw <- data.frame(user = c('Augustin', 'Matt', 'Harvey'), role = c('Manager', 'User', 'User'), password = c('ABC', 'DEF', 'GHI'))
# db.pw$encrypt <- apply(db.pw, 1, function(x) password_store(x['password']))
# db = dbConnect(SQLite(), dbname = 'auth_hash.sqlite')
# dbSendQuery(db, 'CREATE TABLE pw (user TEXT, password TEXT, role TEXT)')
# apply(db.pw, 1, function(x) dbSendQuery(db, paste0('INSERT INTO pw VALUES("', x['user'], '", "', x['encrypt'], '", "', x['role'], '")')))
# dbDisconnect(db)

## Connect to the database (may be a remote connection)
db = dbConnect(SQLite(), dbname = 'auth_hash.sqlite')

server <- function(input, output, session) {
  
  ## Initialize - user is not logged in
  user <- reactiveValues(login = FALSE, name = NULL, role = NULL, header = NULL)
  
  ## Display login modal
  observe({
    showModal(modalDialog(
      title = "Enter Login Details",
      textInput('userInp', 'Login'),
      passwordInput('pwInp', 'Password'),
      actionButton('butLogin', 'Login', class = 'btn action-button btn-success', icon = icon('sign-in')),
      size = 's',
      easyClose = FALSE,
      footer = NULL
    ))
  })
  
  ## Check for user in database
  observeEvent(input$butLogin, {  ## login button pressed
    req(input$userInp, input$pwInp)  ## ensure we have inputs
    removeModal()  ## remove the modal
    pw_out <- dbGetQuery(db, paste0('SELECT password FROM pw WHERE user = \"', input$userInp, '\"'))  ## query database
    if (nrow(pw_out) == 0) {  ## user does not exist
      user$login <- FALSE
      user$header <- 'ERROR - UNKNOWN USER'
    } else {
      pw <- as.character(pw_out$password)[[1]]  ## grab password from database
      passwordVerified <- password_verify(pw, input$pwInp)  ## check that it matches user input
      if (passwordVerified) {  ## match
        user$login <- TRUE
        user$name <- input$userInp
        user$role <- db.pw[db.pw$user == input$userInp, 'role']
        user$header <- paste0(user$name, ' (', user$role, ')')
      } else {  ## no match
        user$login <- FALSE
        user$header <- 'ERROR - INCORRECT PASSWORD'
      }
    }
  })
  
  ## close database on exit
  session$onSessionEnded(function(){
    dbDisconnect(db)
  })
  
  output$data <- renderUI({
    h4(user$header)
  })
  
  output$myPlot <- renderPlot({
    req(user$login)
    if (user$role == 'Manager') {  ## If manager role, display iris plot
      plot(iris$Sepal.Length, iris$Sepal.Width)
    } else {  ## If user role, display mtcars plot
      plot(mtcars$mpg, mtcars$cyl)
    }
  })
  
}

ui <- fluidPage(
  uiOutput('data'),
  plotOutput('myPlot')
)

shinyApp(ui = ui, server = server)
```