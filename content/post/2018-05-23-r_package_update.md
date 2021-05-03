+++
author = "Harvey"
title = "R 3.5.0 Package Update"
date = "2018-05-23"
description = "Working with user and system libraries in R"
tags = ["R"]
+++

After our IT colleagues updated R to version 3.5.0 I found that I have many packages duplicated between the system library and my user library.  This short piece of code lists all packages with their version for all libraries.  It helps identify those that can be removed from the user library or updated.

```r
## assess user and system packages
##

library(purrr)
library(dplyr)

df.all <- as.data.frame(installed.packages()[,c(1:4, 16)], stringsAsFactors = FALSE)
df.split <- split.data.frame(df.all, df.all$LibPath)
df.wide <- reduce(df.split, full_join, by = 'Package')
names(df.wide) <- c('Package', paste0(rep(c('libpath_', 'version_', 'priority_', 'built_'), length(df.split)), rep(seq(length(df.split)), each = 4)))

## in library 2 but not 1
df.user_lib <- df.wide %>%
  filter(is.na(libpath_1) & !is.na(libpath_2))

## in both library 1 and 2
df.libs12 <- df.wide %>%
  filter(!is.na(libpath_1) & !is.na(libpath_2))
```