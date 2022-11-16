+++
author = "Harvey"
title = "Shiny Frameworks - Instructions"
date = "2022-11-14"
description = "Shiny Frameworks - Instructions"
tags = ["R", "shiny", "shiny framework"]
codeMaxLines = 15
draft = true
+++

# The Instruction Database / File

Instructions are the soul of the framework.  They can be considered a recipe or lists of steps that are read and executed by an interpreter.  Instructions can be a single sequence of executional steps or, more likely, a collection where each member is a set of steps.  This latter approach leverages the strength of a framework over a traditional app as it can scale very quickly.

Instructions can sit in a number of locations, here we shall discuss a database format and a file format.

## Database format

When working with a collection of instructions I favor a noSQL database approach - each instructional set is separated from the others and can, potentially, rely on different fields.  NoSQL databases based on JSON such as MongoDB and CouchDB are ideally suited for a framework approach as this format can store complex data in a convenient format and work with hierarchical data.  Moreover, there are simple R libraries that can interact with these databases, returning data as a named list.

## File format

Similarly to the database approach, a JSON flat file works well for a framework approach.  The jsonlite library can be used to read the entire file into memory (as a list) prior to processing.

## Database vs File

There are some advantages in considering a database over a flat file format.  Firstly, individual records can be accessed far more readily from a database as compared to a file.  Secondly, databases scale far more readily than files, which tend to require reading the entire structure into memory before processing.  An advantage of the file format is that there is a greater flexibility on where it can be located.  A database generally requires a server, whereas a JSON file can be stored on a shared drive or pinned on a board using the pins library.

## Accessing Instructions

Whether a JSON flat file or JSON-based database is used to store the instructions, it makes sense to define a single set of functions to access the details.  This is a powerful design choice for a framework - the ability to access instructions the same way whether they are stored in a database or in a file.  In the example below we use db_question() function to read a question whether it is stored in a database (via an API call) or a flat file (read into memory).

