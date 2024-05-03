+++
author = "Harvey"
title = "Fun With R6 Classes"
date = "2024-04-28"
description = "child R6 classes that update a parent"
tags = ["R"]
codeMaxLines = 15
draft = true
+++

R has several object-oriented systems and I'm a big fan of R6.  Detailed below is a specific use-case.  I wanted a parent class that held a list of child classes with thet specification that the child class instances could update the parent class instance.

## Parent Class

The parent class is shown below along with a table detailing the public and private fields and methods.  the purpose of the parent class is to hold a series of steps along with methods to interact with them.  In addition, the parent class has a private field called `accumulator` which we will update from the child classes.

| public/private | field/method | description |
| -- | -- | -- |
| public | name | a label |
| public | initialize() | create a new instance |
| public | update(n) | update the accumulator by n (default n = 1) |
| public | count() | return the value of the accumulator |
| public | add(step) | add a new step to the parent class (steps are child classes) |
| public | run() | execute all the steps (child classes) |
| public | status() | return the status of each step |
| private | accumulator | an accumulator, intially set to 0 |
| private | steps | list of steps |

```r
#' R6 parent class
parent_class <- R6::R6Class("parent_class",
                            public = list(

                              #' @field name Class label
                              name = "",

                              #' @description
                              #' Initialize the class
                              initialize = function(name) {
                                self$name = name
                                invisible(self)
                              },

                              #' @description
                              #' Update accumulator by value
                              update = function(n = 1) {
                                private$accumulator <- private$accumulator + n
                              },

                              #' @description
                              #' Return the value of the accumulator
                              count = function() {
                                return(private$accumulator)
                              },

                              #' @description
                              #' Add a new step
                              #' @param step type of step to add
                              add = function(step) {
                                new_name <- paste0(sample(LETTERS, size = 8), collapse = "")
                                new_step <- get(step)$new(name = new_name)
                                private$steps[[new_name]] <- new_step
                              },

                              #' @description
                              #' Run the steps
                              run = function() {
                                for (s in private$steps) {
                                  s$execute(parent = self)
                                }
                              },

                              #' @description
                              #' Return status of steps
                              status = function() {
                                lapply(private$steps, function(s) {
                                  list(name = s$name, value = s$val, status = s$status)
                                }) |> dplyr::bind_rows()
                              }

                            ),

                            private = list(
                              accumulator = 0,
                              steps = list()
                            )
)
```

## Child Class - Generic

For child classes we first build a generic class that can manage any function that is common across the child classes.   We can then use the property of inheritance so that the generic child class methods are available for all child classes, adding any specific methods.  The generic class is shown below along with a list of public fields and methods.

| field/method | description |
| -- | -- |
| name | a label |
| val | numeric to store a class value (intial = NA) |
| status | status notification - possible values are initialized and run |
| initialize() | create a new instance |
| execute() | execute the class - set val equal to parent$count() and change status to run |

```r
child_class <- R6::R6Class("child_class",
                           public = list(

                             #'  @field name class label
                             name = NULL,

                             #' @field val class value
                             val = NA,

                             #' @field status class status
                             status = "initialized",

                             #' @description
                             #' Initialize class
                             initialize = function(name) {
                               self$name <- name
                             },

                             #' @description
                             #' Execute the class.  Set internal value equal to the
                             #'     parent class `accumulator`
                             #' @param parent Parent class
                             execute = function(parent) {
                               self$val <- parent$count()
                               self$status <- "run"
                             }
                           )
)
````

## Child Class - Child Classes

We define two child classes.  The first increases the parent accumulator field by one, and the second doubles it.  Each child class inherits the generic class to avoid repetition.  The only change from the generic class is the public `execute()` method.

| field/method | description |
| -- | -- |
| execute() | execute the class - set val equal to parent$count(), change parent accumulator according to the step, and change status to run |

```r
step_add_one <- R6::R6Class("step_add_one",

                            inherit = child_class,

                            public = list(

                              #' @description
                              #' Execute the class.  Set internal value equal to the
                              #'     parent class `accumulator` and increase the parent
                              #'     class `accumulator` by 1.
                              #' @param parent Parent class
                              execute = function(parent) {
                                self$val <- parent$count()
                                parent$update()
                                self$status <- "run"
                              }
                            )
)
```

```r
step_double <- R6::R6Class("step_double",

                           inherit = child_class,

                           public = list(

                             #' @description
                             #' Execute the class.  Set internal value equal to the
                             #'     parent class `accumulator` and multiply the parent
                             #'     class `accumulator` by 2.
                             #' @param parent Parent class
                             execute = function(parent) {
                               self$val <- parent$count()
                               parent$update(n = parent$count())
                               self$status <- "run"
                             }
                           )
)
```

## Execution

```r
my_parent <- parent_class$new('parent class')
my_parent$add('step_add_one')
my_parent$add('step_add_one')
my_parent$add('step_double')
my_parent$add('step_double')
my_parent$count()
[1] 0

my_parent$status()
# A tibble: 4 × 3
  name     value status     
  <chr>    <lgl> <chr>      
1 UWCITAHO NA    initialized
2 KCSFEWMA NA    initialized
3 ZJYICBPX NA    initialized
4 AZBIUQMJ NA    initialized

my_parent$run()
my_parent$count()
[1] 8

my_parent$status()
# A tibble: 4 × 3
  name     value status
  <chr>    <dbl> <chr> 
1 UWCITAHO     0 run   
2 KCSFEWMA     1 run   
3 ZJYICBPX     2 run   
4 AZBIUQMJ     4 run   
```

Running the code above creates a parent class instance called `my_parent`.  Four steps are added to the parent class (`step_add_one` twice and `step_double` twice).  At this point, the accumulator (`my_parent$count()`) is 0 and `my_parent$status()` shows all steps are `initialized` as no steps have been executed.  After `my_parent$run()` is run and all steps executed, the accumulator is 8 (add 1, add 1, double, double) and `my_parent$status()` shows all steps are `run`.

The accumulator is a field in the parent and it is updated through the child classes.
