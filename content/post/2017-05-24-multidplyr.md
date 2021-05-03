+++
author = "Harvey"
title = "parallel dplyr"
date = "2017-05-24"
description = "Multidplyr - A parallel version of dplyr"
tags = ["R"]
+++

## Introduction
I’m a big fan of the R packages in the tidyverse and dplyr in particular for performing routine data analysis.  I'm currently working on a project that requires fitting dose-repsonse curves to data.  All works well when the amount of data is small but as the dataset grows so does the computational time.  Fortunately there’s a library (multidplyr) to perform dplyr operations under parallel conditions.  By way of example we’ll create a set of dummy data and compare curve fitting using dplyr and multidplyr.

The dataset used is the spinach dataset that comes with the drc package.  It's a data frame containing 105 observations in 5 groups.  Each group consists of 7  concentrations run in triplicate.  To compare dplyr and multidplyr we'll take these measurements and copy them 1000 times with some jitter.

## Code

```r

## Libraries
library(drc)
library(dplyr)
library(multidplyr)
library(microbenchmark)
library(ggplot2)
library(tidyr)

## Create a dummy dataset
data <- spinach
for (i in 1:1000) {
  addData <- spinach
  addData$CURVE <- addData$CURVE + 5 * i
  addData$SLOPE <- sapply(addData$SLOPE, function(x) jitter(x, factor = 10))
  data <- rbind(data, addData)
}

## Define some functions
makeFit <- function(d) {
  tryCatch(drm(SLOPE ~ DOSE, data = d, fct = LL.4()), error = function(e) NA)
}

fit_dplyr <- function(data, n) {
  data %>%
    filter(CURVE <= n) %>%
    group_by(CURVE) %>%
    do(fit = makeFit(.))
}

fit_multidplyr <- function(data, n) {
  data %>%
    filter(CURVE <= n) %>%
    partition(CURVE) %>%
    cluster_copy(makeFit) %>%
    cluster_library('drc') %>%
    do(fit = makeFit(.)) %>%
    collect(unique_indexes = 'CURVE')
}

## Benchmark our data
microbenchmark(fit_dplyr(data, 10), times = 3)
microbenchmark(fit_dplyr(data, 100), times = 3)
microbenchmark(fit_dplyr(data, 1000), times = 3)
microbenchmark(fit_dplyr(data, 5000), times = 3)
microbenchmark(fit_multidplyr(data, 10), times = 3)
microbenchmark(fit_multidplyr(data, 100), times = 3)
microbenchmark(fit_multidplyr(data, 1000), times = 3)
microbenchmark(fit_multidplyr(data, 5000), times = 3)

## Conclude with a table and graph
df.graph <- data.frame(n = rep(c(10, 100, 1000, 5000), 2), library=rep(c('dplyr', 'multidplyr'), each=4), timing=c(0.20, 3.04, 39.07, 212.89, 0.13, 1.13, 10.13, 49.29))

ggplot(df.graph, aes(x=n, y=timing, colour=library)) +
  geom_point(size = 2) +
  geom_line() +
  labs(x = 'number of groups', y = 'timing (seconds)')

df.table <- df.graph %>%
  spread(library, timing) %>%
  mutate(enhancement = round(dplyr / multidplyr, 2))

```

## Conclusion

In this case, multidplyr runs up to 4.3 times faster on a 16 core PC.  The speed enchancement increases with increasing size of the dataset.


|  n   | dplyr (secs) | multidplyr (secs) |
|------|--------------|-------------------|
| 10   | 0.20         |  0.13             |
| 100  | 3.04         |  1.13             |
| 1000 | 39.07        |  10.13            |
| 5000 | 212.89       |  49.29            |

{{< figure src="/images/post-images/2017-05-24-multidplyr/multidplyr.png" >}}
