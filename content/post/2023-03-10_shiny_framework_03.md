+++
author = "Harvey"
title = "Shiny Frameworks 03 - A Simple Interpreter"
date = "2023-03-10"
description = "Shiny Frameworks - A Simple Interpreter"
tags = ["R", "shiny", "shiny framework"]
codeMaxLines = 15
draft = true
+++

## Introduction

Continuing the series on [shiny frameworks](/tags/shiny-framework/), this post will cover the concept of interpreters with a simple example.

## Shiny Framework Interpreters

Where [instructions](/post/2023-03-03_shiny_framework_02/) are the soul of a shiny framework, interpreters are the heart.  An interpreter takes input instructions, parses them and generates an output.  An ideal interpreter should be data agnostic, meaning that it can take multiple, different types of data and work with them accordingly.  The instructions tell the interpreter what to do and how to handle the data.

In one simple example below we'll read in an instruction file and parse its contents to build a plot.

The instructional file (`instructions.json`) looks like this:

```json
{
  "build": "plot",
  "data": "mtcars",
  "type": "scatter",
  "x": "mpg",
  "y": "wt"
}
```

The interpreter is a parser that can read this input and act upon it.

```r
## import json to R list
instr <- jsonlite::fromJSON("instructions.json", simplifyVector = FALSE)

## parse contents and build plot
if (isTRUE(instr$build == "plot")) {
  data <- get(instr$data)
  if (isTRUE(instr$type == "scatter")) {
    plot(data[[instr$x]],
         data[[instr$y]],
         xlab = instr$x,
         ylab = instr$y,
         type = "p")
  }
}
```

Upon running we get a plot that has been built using the parameters from the json file.

{{< figure src="/images/post-images/2023-03-10-shiny_framework_03/scatter.png" >}}

Unpacking the code above we first import the json file using the `jsonlite` library.  The file will be interpreted as a list by R:

```r
> instr
$build
[1] "plot"

$data
[1] "mtcars"

$type
[1] "scatter"

$x
[1] "mpg"

$y
[1] "wt"
```

The parser works by first identifying if `instr$build` is `plot` and then generating the plot.  We use `isTRUE` to check the value of `instr$build` as it returns `false` if there is no match or if the list `instr` does not have an element named `build`.  The parser then assigns the data to a variable using the `get` function and builds a scatterplot, assigning parameters as applicable.  
This simple example does not include error checking for missing parameters.  The parser would fail if, for example, `x` or `y` were missing or referenced a column in the data that does not exist.  This can be easily mitigated by validating inputs or using `tryCatch` to handle errors.  
From this simple example it's easy to see how our parser can be extended:

```r
## import json to R list
instr <- jsonlite::fromJSON("instructions.json", simplifyVector = FALSE)

## parse contents and build plot
if (isTRUE(instr$build == "plot")) {
  data <- get(instr$data)
  if (isTRUE(instr$type == "scatter")) {
    plot(data[[instr$x]],
         data[[instr$y]],
         xlab = instr$x,
         ylab = instr$y,
         type = "p")
  } else if (isTRUE(instr$type == "line")) {
    plot(data[[instr$x]],
         data[[instr$y]],
         xlab = instr$x,
         ylab = instr$y,
         type = "l")
  }
}
```

and simplified:

```r
## import json to R list
instr <- jsonlite::fromJSON("instructions.json", simplifyVector = FALSE)

## list of plot types
plot_type <- list(scatter = "p", line = "l", both = "b")

## parse contents and build plot
if (isTRUE(instr$build == "plot")) {
  data <- get(instr$data)
  plot(data[[instr$x]],
        data[[instr$y]],
        xlab = instr$x,
        ylab = instr$y,
        type = plot_type[[instr$type]])
}
```

This post introduced the concept of an interpreter, or parser, with a simple example.
