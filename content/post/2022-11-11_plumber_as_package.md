+++
author = "Harvey"
title = "plumber API as a package"
date = "2022-11-11"
description = "Building a plumber API as an R package"
tags = ["R", "plumber", "RSConnect"]
codeMaxLines = 15
draft = false
+++

# Why package a plumber API?

There are many advantages to building R-based code as a package.  You can use the DESCRIPTION file to store metadata related to the API, separate functions from endpoints in the `R/` folder, include documentation and run tests.  
In this example we'll build a simple plumber API as a package.  The API will have two endpoints - `/version` will return the package version and `/sum` will sum two numbers.

# Package folder structure and files

This is the package folder structure.  It follows a typical R package structure but includes an `entrypoint.R` file in the root folder.

```
genericAPI
├── DESCRIPTION  
├── entrypoint.R  
├── LICENSE  
├── LICENSE.md
├── man  
├── NAMESPACE  
├── R  
│   ├── plumber.R  
│   └── sum_numbers.R  
├── tests  
    ├── testthat.R  
    └── testthat  
        └── test-sum_numbers.R  
```

## DESCRIPTION, NAMESPACE and license.md

DESCRIPTION is a typical DESCRIPTION file, as shown below.

```R
Package: genericAPI
Title: A simple generic API
Version: 0.0.1
Authors@R: 
  person("Harvey", "Lieberman", "harvey.lieberman@novartis.com", role = c("aut", "cre"))
Description: An R package to demonstrate developing a plumber API as a package.
License: MIT + file LICENSE
Encoding: UTF-8
Imports:
  pkgload,
  plumber
RoxygenNote: 7.2.1
Suggests: 
    testthat (>= 3.0.0)
Config/testthat/edition: 3
```

NAMESPACE is built using `roxygen2::roxygenise()`.

```r
# Generated by roxygen2: do not edit by hand

import(pkgload)
import(plumber)
```

## entrypoint.R

When starting an API, the plumber package looks for `plumber.R` to parse as the plumber router definition.  Alternatively, if an `entrypoint.R` file is found it will take precedence and be responsible for returning a runnable router.  
The `entrypoint.R` is simple.  It loads all function definitions and then points to the `plumber.R` file under the `R/` folder.

```r
## load all functions
pkgload::load_all()

## start plumber
pr <- plumber::plumb("./R/plumber.R")
```

## R/plumber.R

There is no difference between this and the `plumber.R` file in a traditional plumber API.  It holds the endpoints for the API.  One advantage of building a package, however, is that the functions can be separated into other files.  This makes the `plumber.R` file more succint, holding just the API endpoints.

In the code below we include a `NULL` function so that any packages required by the API are added to `NAMESPACE`.  The `/version` endpoint returns the package version and the `/sum` endpoint calls a function called `sum_numbers()`.

```r
#* @apiTitle My generic plumber API

#' plumber functions
#' 
#' plumber endpoints
#' 
#' @name plumber
#' @import pkgload
#' @import plumber
NULL

#* API version
#* 
#* return an API version
#* 
#* @serializer unboxedJSON
#* 
#* @get /version
function() {
  list(version = as.character(packageVersion("genericAPI")))
}

#* Sum
#* 
#* Sum two numbers
#* 
#* @param a a number
#* @param b a number
#* 
#* @serializer unboxedJSON
#* 
#* @get /sum
function(a, b) {
  list(sum = sum_numbers(a, b))
}
```

## R/sum _ numbers.R

`sum_numbers()` is a simple function returning the sum of two numbers.  Since it's included in a package we can take advantage of roxygen2 documentation.  As it is only used within the API, this function does not have to be exported.

```r
#' my simple function
#' 
#' A simple addition function
#' 
#' @param a numeric
#' @param b numeric
#' 
sum_numbers <- function(a, b) {
  tryCatch({
    as.numeric(a) + as.numeric(b)
  }, 
  warning = function(e) {
    "not numeric"
  },
  error = function(e) {
    "error"
  })
}
```

## testthat/test-sum _ numbers.R

Finally, we can add tests.  The `test-sum_numbers.R` runs three simple tests to ensure our function is performing correctly.

```r
test_that("summation works", {
  expect_equal(sum_numbers('1', '2'), 3)
  expect_equal(sum_numbers(1, 2), 3)
  expect_equal(sum_numbers(1, "A"), "not numeric")
})
```

# Deployment

Deployment to RStudio Connect is simple using the `rsconnect` pacakge

```r
rsconnect::deployAPI(
  api = ".",
  apiTitle = "sample_api",
  appFiles = c("R", "entrypoint.R", "DESCRIPTION", "NAMESPACE"),
  server = **RStudio Connect Server**,
  forceUpdate = TRUE,
  launch.browser = FALSE
)
```

where \*\*RStudio Connect Server\*\* is the URL of the RStudio Connect server.

# Testing the API

In the tests below \*\*url\*\* is the API URL and \*\*apikey\*\* is an RStudio Connect API key

### test 1

```r
out <- httr::GET("**url**/version", httr::add_headers(Authorization = paste("Key", **apikey**)))
httr::content(out)

$version
[1] "0.0.1"
```

### test 2

```r
out <- httr::GET("**url**/sum?a=1&b=4", httr::add_headers(Authorization = paste("Key", **apikey**)))
httr::content(out)

$sum
[1] 5
```

### test 3

```r
out <- httr::GET("**url**/sum?a=1&b=none", httr::add_headers(Authorization = paste("Key", **apikey**)))
httr::content(out)

$sum
[1] "not numeric"
```

# Conclusion

Building a plumber API as an R package provides certain advantages such as documentation, a cleaner folder structure and testing.