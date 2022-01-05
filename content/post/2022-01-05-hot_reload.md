+++
author = "Harvey"
title = "Shiny hot reload"
date = "2022-01-05"
description = "Hot reload when working with a parameter file"
tags = ["R", "shiny"]
draft = false
+++

Here's something I've discovered that may save a huge amount of time in future development.  I'm working on a shiny app that uses reads parameters from a json file and generates a series of plots from the data.  It would be great to use a hot reload during the development phase so that edits to the json file can be visualized without restarting the app each time.  This can be done by setting an option before running the app the first time with this simple line:
```r
options(shiny.autoreload.pattern = glob2rx("*.json"))
```
Now any time a json file is changed the app will automatically reload.  Magic!
