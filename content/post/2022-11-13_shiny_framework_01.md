+++
author = "Harvey"
title = "Shiny Frameworks - Introduction"
date = "2022-11-13"
description = "Shiny Frameworks - Introduction"
tags = ["R", "shiny", "shiny framework", "opinion"]
codeMaxLines = 15
draft = true
+++

## Shiny Frameworks

This is the first in a series of blog posts tagged "shiny framework".  The goal is to provide a some loosely connected articles related to building shiny frameworks with the ultimate goal of turning them into an eBook.

### What is a shiny framework?

Imagine you've built a killer shiny app and you need to change the data.  Most apps are adaptable to a degree.  If you want to upload a different set of data, the app should be able to accommodate if the data are in an expected format.  
But what if:

- your data are in a completely different format?
- you want to manipulate your data in some way (filter or add a new variable)?
- you want to use a completely different visualization?

It's unlikely that the app can cope with these situations unless choices were made at the design stage.

A shiny framework is an approach to building shiny apps which allows for:

- flexibility: being able to offer choices at the populate-stage (more on this below).
- extensibility: ability to readily extend the framework for unmet needs.
- scalability: easily scale beyond the current needs of the app.

### What's the populate-stage?

#### Scenario One

Generally when we think of app development we consider three stages:

- design stage.  Make choices as to how the app should look and behave.
- build stage.  Incorporate choices made at the design stage through coding.
- deployment stage.  Deploy the app for users, gather feedback and respond accordingly.

#### Scenario Two

When we consider a framework approach we can include a fourth stage, populate, in which we populate the framework with information.

- design stage.  Make choices as to how the app should look and behave.
- build stage.  Incorporate choices made at the design stage through coding.
- populate stage.  Populate our app with information.
- deployment stage.  Deploy the app for users, gather feedback and respond accordingly.

In scenario one the developer builds the app and presents it to the customer.  Agile development allows a fast turnaround time but we still have the interaction between the development team and the end-user.

In scenario two we introduce a third person, the knowledge worker.  The knowledge worker is not necessarily the app consumer - they may not be using the app, nor are the developer - they do not create the app.  They are the person who holds the knowledge for the content of the app.  Since they are not a developer they need a way to get information into the app without a programming background.  This is the *populate stage*.  Now, in scenario two, the developer builds a framework for the knowledge worker, the knowledge worker populates it with information and the consumer runs the app.  The knowledge worker is the customer of the developer and the consumer is the customer of the knowledge worker.  The knowledge worker can request changes to the framework from the development team and the consumer can request changes to the app look-and-feel from the same development team.  
In effect, the knowledge worker is involved in a secondary build stage; they are changing the way the app behaves without changing the underlying code.  
This empowers the knowledge worker to have influence how the app behaves without the need of any programming knowledge.

### Sample data

In the related posts we'll build a simple shiny framework which allows someone to create a series of visualizations based on the ferris wheel data taken from Tidy Tuesday (August 9, 2022).
