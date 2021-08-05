+++
author = "Harvey"
title = "Using tidyselect in your own package"
date = "2021-07-22"
description = "Using tidyselect in your own package"
tags = ["R"]
draft = false
+++

The tidyselect package powers column selection for {dplyr}, {tidyr} and {recipes} functions but it's also quite straight-forward to include its functonality in other user-built functions.  Here's an example which replaces a percentage of a data frame by `NA`.  It works by using the `tidyselect::eval_select` function to select columns and then replaces a portion of the data with `NA` values. 

```r
#' populate data frame with missing data
#'
#' Replace a proportion of data in a data frame with missing values
#'
#' @param data A data frame
#' @param p proporion (between 0 and 100) of data in column to be flagged as missing
#' @param ... `<tidy-select>` One or more unquoted expressions separated by commas
#'
#' @return data frame
#'
#' @importFrom rlang expr
#' @importFrom tidyselect vars_select
#' @export
add_missing_df <- function(data, ..., p = 10) {
  expr <- rlang::expr(c(...))
  pos <- tidyselect::eval_select(expr, data = data)
  if (length(pos) > 0) {
    for (posn in pos) {
      missing_rows <- sample(nrow(data), size = as.integer(nrow(data) * p / 100))
      if (length(missing_rows) > 0) {
        data[missing_rows, posn] <- NA
      }
    }
  }
  return(data)
}
```

All of the {tidyselect} selectors may be used, along with the {magrittr} pipe if loaded.  Some examples of use are:
```r
# remove 40% of data in columns mpg to hp
add_missing_df(mtcars, p=40, mpg:hp)
```

```r
# remove 10% of data in columns that start with "d" (dist and drat)
mtcars %>% add_missing_df(startswith("d"))
```