+++
author = "Harvey"
title = "Building Dynamic Nested Loops in R"
date = "2024-08-07"
description = "Building Dynamic Nested Loops in R"
tags = ["R"]
codeMaxLines = 15
draft = true
+++

## Introduction

`lapply` is a fundemental programming function in R, used to apply a function over a list ot a vector.  Running lapply in a nested manner leads to a nested outcome.  For example, the following code:

```r
lapply(c("A", "B"), function(x) {
  lapply(c("C", "D"), function(y) {
    paste0(x, y, collapse = "")
  })
})
```

produces the following output:

```r
[[1]]
[[1]][[1]]
[1] "AC"

[[1]][[2]]
[1] "AD"


[[2]]
[[2]][[1]]
[1] "BC"

[[2]][[2]]
[1] "BD"
```

There are situations where the inputs to the apply function may not be known in advance.  Furthermore, we may not know how deep to nest.  The following code uses a recursive approach to apply functions, returning output from a nested function without having to know how deep the nesting goes.

## Coding

We'll start with the calling function, called `run_loops()`.

```r
#' run_loops
#' Run a series of nested lapply functions, returning a nested
#' or flattened list
#' 
#' @param vals list of lists or vectors to run lapply over
#' @param fn calling function
#' @param flatten if TRUE then flatten the resulting output
#' @param ... other named parameters passed to calling function
#' 
run_loops <- function(vals, fn, flatten = FALSE, ...) {
  rtn <- .build_loop(vals = vals, params = list(), fn = fn, ...)
  if (isTRUE(flatten)) {
    return(.flatten(rtn, level = length(vals) - 1))
  } else {
    return(rtn)
  }
}
```

In this function, `vals` is a list of vectors to be called.  To mimic the `lapply()` example above, `vals = list(a = c("A", "B"), b = c("C", "D"))`.  `fn` is the name of the function to call within the innermost lapply loop.  `flatten` is a boolean specifying whether the resulting output should be flattened or kept as a nested list. `...` allows additional parameters to be passed to `fn`.  The function simply calls the recursive function `.build_loop()` with starting values and either returns the output or the flattened output once recursion has completed.

Next comes the main recursive function, called `.build_loop()`.  

```r
#' .build_loop
#' Add an lapply function or run the calling function
#' 
#' @param vals list of lists or vectors to run lapply over
#' @param params current list of parameters for calling function
#' @param fn calling function
#' @param ... other named parameters passed to calling function
#' 
.build_loop <- function(vals = list(), params = list(), fn, ...) {
  if (length(vals) > 0) {
    lapply(vals[[1]], function(x) {
      .build_loop(vals = vals[-1], params = c(params, setNames(x, names(vals)[1])), fn, ...)
    })

  } else {
    return(fn(params, ...))
  }
}
```

`.build_loop()` takes a number of parameters.  `vals`, `params` and `...` are initially passed from `run_loops()`.  `params` is a named list of parameters, passed to `fn`.  It's worth spending a little time understanding this function as it powers the approach.  Let's provide an example and start to work through the logic.

```r
run_loops(vals = list(a=c('a', 'b', 'c'), b=c('d', 'e'), c=c('f', 'g')), fn = f1)
```

`run_loops()` passes `vals` and `fn` to `.build_loop()` as initial values.  `.build_loop()` checks the length of `vals` (in this case 3) and, if it's greater than zero, continues.  We then construct an lapply function to run over all values of `vals[[1]]`, which, in this case is *c('a', 'b', 'c')*.  Within the lapply function we recursively call `.build_loop()` on shorter and shorter iterations of `vals`, removing the first group each time and longer and longer iterations of `params`, the named list of parameters.  
the recursive function's terminal condition exists when `vals` has been reduced to an empty list and `params` contains a named list of parameters to pass to the calling function `fn`.  The function `fn` runs on the named parameters, `params` which, in the first call will be `c(a='a', b='d', c='f')`, on the second call will be `c(a='a', b='d', c='g')`, and so on.

The final function is another recursive function, called `.flatten()`.  This function recursively flattens a list to a specified level.

```r
#' .flatten
#' Recursively flatten a list
#' 
#' @param l list
#' @param level level to flatten to
#'
.flatten <- function(l, level = 1) {
  if (purrr::pluck_depth(l) > level) {
    .flatten(purrr::list_flatten(l), level)
  } else {
    l
  }
}
```

## Execution

`run_loops()` can be called as follows:

```r
#' f1
#' Function to test recursive lapply
#' Paste a series of characters with an optional prefix
#' 
f1 <- function(params, ...) {
  args <- list(...)
  paste0(c(args[["prefix"]], params), collapse = "")
}

run_loops(vals = list(a=c('a', 'b', 'c'), b=c('d', 'e'), c=c('f', 'g')), fn = f1, flatten = FALSE, prefix = "PRE_")
```

In this example, we request a nested structure at three levels.  For each combination we'll run a function, `f1`.  The lapply function has three levels of nesting with 3, 2, and 2 parameters, leading to 3x2x2 = 12 outputs.  Running the function as written above returns the following:

```r
[[1]]
[[1]][[1]]
[[1]][[1]][[1]]
[1] "PRE_adf"

[[1]][[1]][[2]]
[1] "PRE_adg"


[[1]][[2]]
[[1]][[2]][[1]]
[1] "PRE_aef"

[[1]][[2]][[2]]
[1] "PRE_aeg"



[[2]]
[[2]][[1]]
[[2]][[1]][[1]]
[1] "PRE_bdf"

[[2]][[1]][[2]]
[1] "PRE_bdg"


[[2]][[2]]
[[2]][[2]][[1]]
[1] "PRE_bef"

[[2]][[2]][[2]]
[1] "PRE_beg"



[[3]]
[[3]][[1]]
[[3]][[1]][[1]]
[1] "PRE_cdf"

[[3]][[1]][[2]]
[1] "PRE_cdg"


[[3]][[2]]
[[3]][[2]][[1]]
[1] "PRE_cef"

[[3]][[2]][[2]]
[1] "PRE_ceg"
```

The function can also be run, returning a flattened list as follows:

```r
run_loops(vals = list(a=c('a', 'b', 'c'), b=c('d', 'e'), c=c('f', 'g')), fn = f1, flatten = TRUE, prefix = "PRE_")
```

```r
[[1]]
[1] "PRE_adf"

[[2]]
[1] "PRE_adg"

[[3]]
[1] "PRE_aef"

[[4]]
[1] "PRE_aeg"

[[5]]
[1] "PRE_bdf"

[[6]]
[1] "PRE_bdg"

[[7]]
[1] "PRE_bef"

[[8]]
[1] "PRE_beg"

[[9]]
[1] "PRE_cdf"

[[10]]
[1] "PRE_cdg"

[[11]]
[1] "PRE_cef"

[[12]]
[1] "PRE_ceg"
```

## Alternative method
An alternative approach would be to build all combinations of parameters and then send them to the calling function, `f1`.  The base R function `expand.grid()` can be used to run in this manner.

```r
g <- expand.grid(list(a=c('a', 'b', 'c'), b=c('d', 'e'), c=c('f', 'g')))
apply(g, 1, f1, prefix = "PRE_")
```

```r
 [1] "PRE_adf" "PRE_bdf" "PRE_cdf" "PRE_aef" "PRE_bef" "PRE_cef" "PRE_adg"
 [8] "PRE_bdg" "PRE_cdg" "PRE_aeg" "PRE_beg" "PRE_ceg"
 ```

The advantage of `expand.grid()` is that it is far simpler in nature, however the nested structure is lost.  The recursive approach outlined above offers additional advantages when working with more complex logic.
