+++
author = "Harvey"
title = "Restarting a Shiny App"
date = "2018-11-07"
description = "Forcing a shiny app restart"
tags = ["R", "shiny"]
+++

I have to post this for my own sanity.  I tend to create shiny apps in a specific folder and link to them using symbolic links from the ShinyApp folder.  This allows multiple versions (prod and dev) and keeps code clean.  I don't know how many times I've wrestled with an app pointing to an earlier version when updates have been made and it's always driven me crazy.  I also don't know how many times I've read through the shiny-server documentation but never discovered section 3.1.1 - Restarting an Application.  Now I'm happy I've discovered `touch restart.txt` as a way to force an app to restart.  All my versioning issues have gone away.
