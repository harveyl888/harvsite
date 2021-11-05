+++
author = "Harvey"
title = "Multiwindow App"
date = "2021-08-30"
description = "Building a multiwindow shiny app"
tags = ["R", "shiny", "javascript"]
draft = true
+++

This is an initial concept of a framework for multiwindow shiny apps.  Shiny is a great R package for building an interactive, web-based, graphical UI using R.  It does, however, suffer from some limitations.  One of these is the fact that a Shiny app runs in a single browser window.  To make a more complex interface we can extend the browser window to full screen and add tab panels or scrollbars.  Another concept is to break parts of the app into separate apps, each with their own window, separate but linked through a single data source (a single source of truth).  
In this prototype, one app pushes data to a json file and two others read the data from this file using a reactiveFileReader.  The reactiveFileReader ensures that any changes made to the data updates the apps that are dependent upon it.  One additional shiny app (`launcher`) is responsible for opening and closing all others.
