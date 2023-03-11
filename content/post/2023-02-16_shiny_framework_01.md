+++
author = "Harvey"
title = "Shiny Frameworks 01 - Introduction"
date = "2023-02-16"
description = "Shiny Frameworks - Introduction"
tags = ["R", "shiny", "shiny framework", "opinion"]
codeMaxLines = 15
draft = true
+++

## Shiny Frameworks

This is the first in a series of blog posts tagged "shiny framework".  The goal is to provide a some loosely connected articles related to building shiny frameworks with the ultimate goal of turning them into an eBook.  Inspiration comes from a presentation I gave at R/Pharma 2022 (https://youtu.be/bkKe_kC83iQ).

### What is a shiny framework?

Imagine you've built a killer shiny app and you need to change the data.  Most apps are adaptable to a degree.  If you want to upload a different set of data, the app should be able to accommodate if the data are in an expected format.  
But what if:

- your data are in a completely different format?
- you want to manipulate your data in some way (filter or add a new variable)?
- you want to use a completely different visualization?

It's unlikely that the app can cope with these situations unless choices were made at the design stage.

A shiny framework is an approach to building shiny apps which allows for:

- flexibility: being able to offer choices after the app has been built.
- extensibility: ability to readily extend the framework for unmet needs.
- scalability: easily scale beyond the current needs of the app.

The basic premise is to build the app using instructions.  These instructions are parsed through an interpreter, which converts the instructions into R code.  An interpreter can do many things, such as:

- reading an interpreting files of different formats
- data wrangling and manipulation
- building static or dynamic outputs
- buiding and/or responding to shiny reactives

Taken to the extreme, a shiny framework can be used to build any part of an app.  It's a powerful resource to simplify the build for a series of similar apps.  In effect you could build a framework that builds other frameworks or an app that builds other apps.

### The layer concept in shiny frameworks

When working with a framework its a good programming practice to take on a three layer approach:

- Data Layer.  This is where the data reside along with any functions that interact with the data.
- Application Layer.  This is where the data are processed and any analyses are performed.
- Presentation Layer.  This is where the output is generated and presented to the user.

It's important to note that layers can only interact with adjacent layers, meaning that information may pass from the data layer to the application layer and from the application layer to the presentation layer but **NOT** from the data layer directly to the presentation layer - it must pass through the application layer first.

Why is this a good way to work?  Well, firstly it forces you to separate out the tasks.  From an R-perspective it's easier to maintain once you realize that data interaction, processing and presentation can each exist in their own separate packages.  Secondly it allows for simpler scalability - data retrieval and processing can be accomplished using an API running on a different architecture, releasing burden from a shiny app.  Taking this a step further, each of these layers may be a different language (python API to access data, R to process and javascript to display).

{{< figure src="/images/post-images/2023-02-16-shiny_framework_01/layers.png" >}}
