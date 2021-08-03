+++
author = "Harvey"
title = "Passing a Mixture of Reactive and Non-Reactives to a Shiny Module"
date = "2021-06-20"
description = "Passing a Mixture of Reactive and Non-Reactives to a Shiny Module"
tags = ["R", "shiny"]
draft = true
+++

Generally there is no issue in sending a list of parameters (reactive and non-reactive) to a shiny module.  Here's an example where a shiny module would be called multiple times, programatically, where the reactive nature of the parameters may be variable (reactive in one instance but not in another).  One way to deal with this is to read in the list of parameters and convert the non-reactive ones to reactive.  Those originally reactive, remain so and therefore update on a change.

### app.R
```r

```
