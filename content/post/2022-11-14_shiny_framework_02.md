+++
author = "Harvey"
title = "Shiny Frameworks - Separation of Tasks"
date = "2022-11-14"
description = "Shiny Frameworks - Separation of Tasks"
tags = ["R", "shiny", "shiny framework", "opinion"]
codeMaxLines = 15
draft = true
+++

# Separation of Tasks

When working with a framework its a good programming practice to take on a three layer approach:
-  Data Layer.  This is where the data reside along with any functions that interact with the data.
-  Application Layer.  This is where the data are processed and any analyses are performed.
-  Presentation Layer.  This is where the output is generated and presented to the user.

It's important to note that layers can only interact with adjacent layers, meaning that information may pass from the data layer to the application layer and from the application layer to the presentation layer but **NOT** from the data layer directly to the presentation layer - it must pass through the application layer first.

Why is this a good way to work?  Well, firstly it forces you to separate out the tasks.  From an R-perspective it's easier to maintain once you realize that data interaction, processing and presentation can each exist in their own separate packages.  Secondly it allows for simpler scalability - data retrieval and processing can be accomplished using an API running on a different architecture, releasing burden from a shiny app.  Taking this further, each of these layers may be a different language (python API to access data, R to process and javascript to display).

{{< figure src="/images/post-images/2022-11-14-shiny_framework_02/layers.png" >}}
