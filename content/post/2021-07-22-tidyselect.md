+++
author = "Harvey"
title = "Using tidyselect in your own package"
date = "2021-07-22"
description = "Using tidyselect in your own package"
tags = ["R"]
draft = false
+++

The {tidyselect} package powers column selection for {dplyr}, {tidyr} and {recipes} functions but it's also quite straight-forward to include its functonality in other user-built functions.  Here's an example which replaces a percentage of a data frame by `NA`.  It works by using the `tidyselect::eval_select` function to select columns and then replaces a portion of the data with `NA` values.  
Column selections are passed to the function through dot-dot-dot, allowing an arbitrary number of columns to be selected.  The first line of the function: 
```r 
expr <- rlang::expr(c(...))
```
returns a *defused* expression, and the second:
```r 
pos <- tidyselect::eval_select(expr, data = data)
```
resumes execution, returning a vector of positions that match the selection.  More details on how to implement tidyselect interfaces is available at the [tidyselect pkgdown page](https://tidyselect.r-lib.org/articles/tidyselect.html).

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

## Examples

All of the {tidyselect} selectors may be used, along with the {magrittr} pipe if loaded.  Some examples of use are:
```r
# remove 40% of data in columns mpg to hp
df <- add_missing_df(mtcars, p=40, mpg:hp)
head(df)
```
|                  | mpg |cyl |disp | hp |drat |   wt | qsec |vs |am |gear |carb|
|-|-|-|-|-|-|-|-|-|-|-|-|
|Mazda RX4         |21.0 |  6 |  **NA** |  **NA** |3.90 |2.620 |16.46 | 0 | 1 |   4 |   4|
|Mazda RX4 Wag     |21.0 |  **NA** | 160 |  **NA** |3.90 |2.875 |17.02 | 0 | 1 |   4 |   4|
|Datsun 710        |22.8 |  **NA** |  **NA** |  **NA** |3.85 |2.320 |18.61 | 1 | 1 |   4 |   1|
|Hornet 4 Drive    |  **NA** |  **NA** | 258 |110 |3.08 |3.215 |19.44 | 1 | 0 |   3 |   1|
|Hornet Sportabout |  **NA** |  8 |  **NA** |175 |3.15 |3.440 |17.02 | 0 | 0 |   3 |   2|
|Valiant           |18.1 |  **NA** | 225 |105 |2.76 |3.460 |20.22 | 1 | 0 |   3 |   1|

```r
# remove 10% of data in columns that start with "d" (dist and drat)
df <- mtcars %>% add_missing_df(startswith("d"))
head(df)
```
|                  | mpg |cyl |disp | hp |drat |   wt | qsec |vs |am |gear |carb|
|-|-|-|-|-|-|-|-|-|-|-|-|
|Mazda RX4         |21.0 |  6 |  **NA** |110 |  **NA** |2.620 |16.46 | 0 | 1 |   4 |   4|
|Mazda RX4 Wag     |21.0 |  6 | 160 |110 |3.90 |2.875 |17.02 | 0 | 1 |   4 |   4|
|Datsun 710        |22.8 |  4 | 108 | 93 |3.85 |2.320 |18.61 | 1 | 1 |   4 |   1|
|Hornet 4 Drive    |21.4 |  6 | 258 |110 |3.08 |3.215 |19.44 | 1 | 0 |   3 |   1|
|Hornet Sportabout |18.7 |  8 | 360 |175 |  **NA** |3.440 |17.02 | 0 | 0 |   3 |   2|
|Valiant           |18.1 |  6 | 225 |105 |2.76 |3.460 |20.22 | 1 | 0 |   3 |   1|
