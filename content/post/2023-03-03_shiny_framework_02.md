+++
author = "Harvey"
title = "Shiny Frameworks 02 - Instructions and Parsing"
date = "2023-03-03"
description = "Shiny Frameworks - Instructions and Parsing"
tags = ["R", "shiny", "shiny framework", "opinion"]
codeMaxLines = 15
draft = true
+++

## Introduction

Continuing the series on [shiny frameworks](/tags/shiny-framework/), this post will cover the concept of framework instructions, how and where to store them and introduce the concept of interpreters.

## Shiny Framework Instructions

As mentioned in [part 1](/post/2023-02-16_shiny_framework_01/), the concept of a shiny framework is to build an app or its content using instructions.  The instructions are parsed through an interpreter which converts the instructions into code.  Instructions are the soul of the framework.  They can be considered a recipe or lists of steps that are read and executed by an interpreter.  Instructions may be a single sequence of steps which might be used to set up parts of an app, such as defining a consistent UI.  More likely, the instructions are a collection where performs a specific task.  The latter is where the strengths of a framework are realized as this approach is highly flexible and scales quickly compared to a traditional shiny app.

## Instruction Format

The best way to store instructional information is in a json format as shown below

```json
{
  "param_01": "value_01",
  "param_02": "value_02",
  "param_03": "value_03",
}
```

When read into R using the popular {jsonlite} library, the json format is converted into a named list.  When we introduce interpreters you'll see how named lists work very well when parsing instruction parameters.  For a list of instructions we can use a json array:

```json
[
  {
    "ref": 1,
    "param_01": "value_01A",
    "param_02": "value_02A",
  },
  {
    "ref": 2,
    "param_01": "value_01B",
    "param_02": "value_02B",
    "param_03": "value_03B",
  }
]
```

An advantage of this format, when working with a collection of instructions, is that each instruction set can contain different parameters.

## How and Where to Store Instructions

### Flat File

Json is a language-agnostic format designed to be stored in a simple flat file.  For smaller frameworks, this is an ideal way to store instructions.  When working with a flat file, the entire contents are read into memory and converted to a list in R before processing.

### NoSQL Database

When working with a collection of instructions, a NoSQL database works better than a flat file.  Each instruction set is separated from the others and can be imported indepedently.  NoSQL databases based on JSON such as MongoDB and CouchDB are ideally suited for a framework approach as this format can store complex data in a convenient format and work with hierarchical data.  Moreover, there are simple R libraries that can interact with these databases, returning data as a named list.

### Database vs File

There are some advantages in considering a database over a flat file format.  Firstly, individual records can be accessed far more readily from a database as compared to a file.  Secondly, databases scale better than files, which tend to require reading the entire structure into memory before processing.  An advantage of the file format is that there is a greater flexibility on where it can be located.  A database generally requires a server, whereas a JSON file can be stored on a shared drive or pinned on a board using the {pins} library.

## Parsing

Whether instructions are stored in a flat file or database, it makes sense to define a single set of functions to access the details.  The parser or interpreter takes an instruction set and converts it from a series of parameters to a series of functional steps.
