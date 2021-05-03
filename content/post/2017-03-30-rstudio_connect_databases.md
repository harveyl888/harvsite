+++
author = "Harvey"
title = "RStudio Connect and Databases"
date = "2017-03-30"
description = "Accessing a shared space from RStudio Connect"
tags = ["R"]
+++

I’m starting to experiment with RStudio Connect.  I’ve been working on a shiny app that writes to a database and a flexdashboard that can illustrate the output.  When writing this type of approach using Shiny Server, both pieces of code could simply point to the same sqlite database.  This becomes a little more challenging under RStudio Connect.  One solution is to point both pieces of code to a mysql database which resides outside of the RStudio Connect server.  In order to run this way, the *libmysqlclient* library must be installed on the instance running RStudio Connect.  Once deployed, the app can update the database and the flexdashboard can be configured to pool from the database and automatically update at fixed intervals.
