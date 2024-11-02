+++
author = "Harvey"
title = "Testing for a Float in a Vector"
date = "2024-10-15"
description = "How to check if a floating point is in a Vector"
tags = ["R"]
codeMaxLines = 15
draft = false
+++

Equality between floating points is always challening when programming (see https://en.wikipedia.org/wiki/Floating-point_arithmetic#Accuracy_problems).  One way to determine if two numbers are equal is to set a precision.  In this short snippet I create a function (`%noVwithin%`) to determine if a number exists in a vector of floating point numbers.  The function is then vectorized so that it can be used in tidyverse expressions.

## Function - `%noVwithin%`

```r
precision <- 1e-10

`%noVwithin%` <- function(x, y) {
  any(
    sapply(y, function(z) {
      abs(x - z) <= precision
    })
  )
}
```

The function `%noVwithin%` takes two variables, checking to see if `x` is within the vector `y`.  It is similar to the R function `%in%` but works with floating point numbers.  The example below illustrates both `%in%` and `%noVwithin%`

```r
a <- 0.8
b <- 0.4
val <- a + b
print(val %in% c(1.1, 1.2, 1.3))
# [1] FALSE
print(val %noVwithin% c(1.1, 1.2, 1.3))
# [1] TRUE
```

## Vectorizing the Function

Our function works but fails when used in a tidyverse pipe:

```r
d <- tibble::tibble(
  a = seq(0.1, 0.5, 0.1),
  b = seq(1.1, 1.5, 0.1), 
  val = a + b
)

# # A tibble: 5 × 3
#       a     b   val
#   <dbl> <dbl> <dbl>
# 1   0.1   1.1   1.2
# 2   0.2   1.2   1.4
# 3   0.3   1.3   1.6
# 4   0.4   1.4   1.8
# 5   0.5   1.5   2 

myVals <- seq(1, 1.6, 0.2)
# [1] 1.0 1.2 1.4 1.6

d |>
  dplyr::mutate(in_myVals = val %noVwithin% myVals)
# # A tibble: 5 × 4
#       a     b   val in_myVals
#   <dbl> <dbl> <dbl> <lgl>    
# 1   0.1   1.1   1.2 TRUE     
# 2   0.2   1.2   1.4 TRUE     
# 3   0.3   1.3   1.6 TRUE     
# 4   0.4   1.4   1.8 TRUE     
# 5   0.5   1.5   2   TRUE     

```
Here, `val` is identified as present in `myVals` even when it is not.


Vectorizing is simple.  We just pass the function to `Vectorize()`, passing a list of argument names that we wish to vectorize.  In this case we are passing just the `x` variable as `y` is fixed when calling.

```r
`%within%` <- Vectorize(`%noVwithin%`, vectorize.args = "x")
```

Our new function, `%within%`, is the vectorized version.  Running the code above with `%within%` gives the expected result.

```r
d |>
  dplyr::mutate(in_myVals = val %within% myVals)

# # A tibble: 5 × 4
#       a     b   val in_myVals
#   <dbl> <dbl> <dbl> <lgl>    
# 1   0.1   1.1   1.2 TRUE     
# 2   0.2   1.2   1.4 TRUE     
# 3   0.3   1.3   1.6 TRUE     
# 4   0.4   1.4   1.8 FALSE    
# 5   0.5   1.5   2   FALSE    
```

## Using the %in% Function

Running the above code with the base R `%in%` function (which, like many base R functions, is vectorized) in place of `%within%` produces an interesting output:

```r
d |>
  dplyr::mutate(in_myVals = val %in% myVals)
# # A tibble: 5 × 4
#       a     b   val in_myVals
#   <dbl> <dbl> <dbl> <lgl>    
# 1   0.1   1.1   1.2 FALSE    
# 2   0.2   1.2   1.4 FALSE    
# 3   0.3   1.3   1.6 TRUE     
# 4   0.4   1.4   1.8 FALSE    
# 5   0.5   1.5   2   FALSE  
```

Everything is false, as expected, except for `1.6`.  Looking at `val` and `myVals` illustrates why.  

Here are the values of `val` at 20 decimal places:
```r
d$val  |> formatC(digits = 20, format = 'f')
# [1] "1.20000000000000017764" "1.40000000000000013323"
# [3] "1.60000000000000008882" "1.80000000000000026645"
# [5] "2.00000000000000000000"
```

and here are the values stored in the `myVals` vector:
```r
myVals |> formatC(digits = 20, format = 'f')
# [1] "1.00000000000000000000" "1.19999999999999995559"
# [3] "1.39999999999999991118" "1.60000000000000008882"
```
It's interesting to note that both values for 1.6 (d[3, ]$val and myvals[4]) are identical, hence the `%in%` comparison works for 1.6.


## Alternative approaches

### dplyr::rowwise()

The non-vectorized version works when used in conjunction with `dplyr::rowwise()` as `rowwise` computes one row at a time.

```r
d |>
  dplyr::rowwise() |>
  dplyr::mutate(in_myVals = val %noVwithin% myVals)

# # A tibble: 5 × 4
#       a     b   val in_myVals
#   <dbl> <dbl> <dbl> <lgl>    
# 1   0.1   1.1   1.2 TRUE     
# 2   0.2   1.2   1.4 TRUE     
# 3   0.3   1.3   1.6 TRUE     
# 4   0.4   1.4   1.8 FALSE    
# 5   0.5   1.5   2   FALSE    
```

### purrr::map

The `purrr::map()` functions can work with non-vectorized functions within a `mutate()`.

```r
d |>
  dplyr::mutate(in_myVals = purrr::map_lgl(val, `%noVwithin%`, myVals))

# # A tibble: 5 × 4
#       a     b   val in_myVals
#   <dbl> <dbl> <dbl> <lgl>    
# 1   0.1   1.1   1.2 TRUE     
# 2   0.2   1.2   1.4 TRUE     
# 3   0.3   1.3   1.6 TRUE     
# 4   0.4   1.4   1.8 FALSE    
# 5   0.5   1.5   2   FALSE    
```
