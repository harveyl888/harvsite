+++
author = "Harvey"
title = "Scraping for Dishwashers"
date = "2016-11-24"
description = "Scraping Home Depot to gather dishwasher information (using rvest)"
tags = ["R"]
+++

Our dishwasher broke and was beyond repair.  Time for a new one.  Fortunately Black Friday was approaching so there were plenty of deals to be found.  It seems it's now easier than ever to pull information off the web and much of it is in a usable format so, if you ask me to identify a replacement dishwasher, web-scraping is where I'm going to start.  I'm a big fan of the R library rvest and it took little time to pull back some useful information off the Home Depot website.

```r
library(rvest)
library(xml2)

## Choose a starting point
baseurl <- 'http://www.homedepot.com/b/Appliances-Dishwashers-Built-In-Dishwashers/N-5yc1vZc3nj'

## Data frame to hold results
df <- data.frame(model = character(0), rating = numeric(0), link = character(0))
url <- baseurl
loadNextPage <- TRUE
while(loadNextPage) {  ## Loop through pages
  print('Reading Page')
  Sys.sleep(0.1)  ## Let's be nice
  html <- url %>%
    read_html()  ## pull back the page
  dw <- html %>%
    html_nodes('.plp-pod') ## focus in on the dishwashers
  model <- dw %>%
    html_node('.pod-plp__model') %>%
    html_text() ## get model ID
  model <- gsub('[^[:alnum:] ]', '', model)
  model <- trimws(sub("Model\\s([^ ]*).*$", "\\1", model)) ## remove the unwanted
  rating <- dw %>%
    html_nodes('.pod-plp__ratings') %>%
    html_node('a') %>%
    html_node('span') %>%
    html_attr('rel') %>%
    as.numeric() ## rating can be found in a link
  link <- dw %>%
    html_nodes('.plp-pod__image') %>%
    html_nodes('a') %>%
    html_attr('href') ## link to more information
  df <- rbind(df, data.frame(model = model, rating = rating, link = paste0('http://www.homedepot.com', link)))
  gotoNext <- html %>%
    html_nodes('.hd-pagination__link') %>%
    html_nodes(xpath = '//a[contains(@title,"Next")]') ## Link to the next page
  if (length(gotoNext) > 0) {
    url <- gotoNext %>% html_attr('href')
    url <- paste0('http://www.homedepot.com', url)
    loadNextPage <- TRUE  ## Next page exists
  } else {
    loadNextPage <- FALSE ## We've reached the last page
  }
}
```