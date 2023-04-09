+++
author = "Harvey"
title = "Shiny Frameworks 04 - Importing Data"
date = "2023-03-15"
description = "Shiny Frameworks - Importing Data"
tags = ["R", "shiny", "shiny framework"]
codeMaxLines = 15
draft = true
+++

## Introduction

Continuing the series on [shiny frameworks](/tags/shiny-framework/), this post will cover some concepts of importing data.

## Import data once or many?

Data may be imported when an app starts up or imported as needed.  Depending on the type and size of data, data import may be a slow step and it is prudent to consider the type and size of data required by a shiny framework at the design stage.  Let's consider the two options of importing all data at the start or only importing when data are required.

### Import data once at the start

It makes sense to import data just once, when the app starts up, if the framework relies on only a few data sources.  This occurs when the framework defines a single instruction set or if we have a collection of instructions but these instructions only access a limited number of datasets.

### Import data as needed

Importing data as needed makes sense when you have a collection of instructions that access a large number of datasets - for example if each instruction set accesses a different dataset.  This is particularly true if instruction sets are selected by a user.  For example, consider an app with 200 instructions and outputs where the user may choose which one to process and display.  In such a case we should not import all 200 datasets upfront as only one is required by the app.  The dataset should be imported once the user has selected the instruction set.

## Data import packages

Data may come in many formats.  For now, let's consider CSV since it is a popular format to store rectangular data.  The base R CSV reader function `read.csv()` is relatively slow.  There are faster CSV file readers that may be used in its place such as `read_csv()` from {readr} and `fread` from {data.table}.

To test speed we'll create two CSV files.  `f1.csv` contains 10000 rows of 10 columns of data (first three columns are character and rest are numeric).  `f2.csv` contains 10 rows of 10000 columns of data (first three columns are character and rest are numeric).  `f1.csv` simulates long data whereas `f2.csv` simulates wide.

```r
## create two CSV files
## f1.CSV - 10000 rows, 10 cols, long format
## f2.CSV - 10 rows, 10000 cols, wide format
d <- list(c(10000, 10), c(10, 10000))
for (i in seq_along(d)) {
  f <- sprintf("f_%02i.csv", i)
  data <- matrix(runif(d[[i]][1] * d[[i]][2]), ncol = d[[i]][2]) |>
    data.frame() |>
    dplyr::mutate(dplyr::across(c("X1", "X2", "X3"), as.character))
  write.csv(data, f, row.names = FALSE)
}
```

We can test the speed of `base::read.csv`, `readr::read_csv` and `data.table::fread` using {microbenchmark} and visualize the output with `ggplot::autoplot` as follows

```r
out <- lapply(seq_along(d), function(i) {
  f <- sprintf("f_%02i.csv", i)
  microbenchmark::microbenchmark(
    `read.csv` = read.csv(f),
    read_csv = readr::read_csv(f, show_col_types = FALSE),
    fread = data.table::fread(f), times = 10
  )
})

ggplot2::autoplot(out[[1]])
ggplot2::autoplot(out[[2]])
```

### Benchmark reading long-format CSV
{{< figure src="/images/post-images/2023-03-15-shiny_framework_04/timing_long.png" >}}

### Benchmark reading wide-format CSV
{{< figure src="/images/post-images/2023-03-15-shiny_framework_04/timing_wide.png" >}}

{{% notice note "Note" %}}
Long format relative timing: fread: 1, read_csv: 5.3, read.csv: 13.2  
Wide format relative timing: fread: 1, read_csv: 197, read.csv: 626  

-  `data.table::fread` is the most efficient at reading both long and wide formatted CSV data.
-  reading wide-formatted files is much slower than long-formatted files
{{% /notice %}}

## Importing what is required
Both `readr::read_csv` and `data.table::fread` include a parameter to limit the columns imported.  If you know that not all columns are required, limiting the imported columns can speed up the time.  In the example below we read in just 6 columns.

```r
out_select <- lapply(seq_along(d), function(i) {
  f <- sprintf("f_%02i.csv", i)
  chosen_cols <- c("X1", paste0("X", sample(x = seq(d[[i]][2])[-1], size = 5)))
  microbenchmark::microbenchmark(
    `read.csv` = read.csv(f),
    read_csv = readr::read_csv(f, show_col_types = FALSE, col_select = all_of(chosen_cols)),
    fread = data.table::fread(f, select = chosen_cols), times = 10
  )
})

ggplot2::autoplot(out_select[[1]])
ggplot2::autoplot(out_select[[2]])
```

### Benchmark reading long-format CSV (selected columns)
{{< figure src="/images/post-images/2023-03-15-shiny_framework_04/timing_long_select.png" >}}

### Benchmark reading wide-format CSV (selected columns)
{{< figure src="/images/post-images/2023-03-15-shiny_framework_04/timing_wide_select.png" >}}

{{% notice note "Note" %}}
Long format relative timing: fread: 1, read_csv: 4.8, read.csv: 13.6  
Wide format relative timing: fread: 1, read_csv: 3.1, read.csv: 653  

-  data.table::fread is the most efficient at reading both long and wide formatted CSV data.
-  There is little advantage when selecting columns for `data.table::fread` but when selected with `readr::read_csv` there is a significant increase in speed (1.2x for long data and 65x for wide data)
{{% /notice %}}

## Importing data

Working with data import instructions is relatively simple.  Consider the instructional json file (`instructions.json`):

```json
{
  "instruction": "import",
  "file_name": "my_file.csv"
}
```

The interpreter can be a few lines of code as simple as:

```r
## import json to R list
instr <- jsonlite::fromJSON("instructions.json", simplifyVector = FALSE)

## parse json
if (instr$instruction == "import") {
    my_file <- instr$file_name
    out <- readr::read_csv(my_file, col_types = readr::cols())
}
```

This is perhaps the simplest example.  The interpreter can, however, be more complex depending on needs.  The example below includes an optional file path, identifies the file type (either CSV or SAS) by its extension and allows the import to be limited to a subset of columns.

```r
## import json to R list
instr <- jsonlite::fromJSON("instructions.json", simplifyVector = FALSE)

## parse json
if (instr$instruction == "import") {
  if (!is.null(instr$file_path)) {
    my_file <- file.path(instr$file_path, instr$file_name)
  } else {
    my_file <- instr$file_name
  }
  only_cols <- unlist(instr$cols)

  if(tools::file_ext(my_file) == "csv") {
    out <- readr::read_csv(my_file, col_types = readr::cols(), col_select = only_cols)
  } else if (tools::file_ext(my_file) == "sas7bdat") {
    out <- haven::read_sas(my_file, col_select = only_cols)
  } else {
    out <- data.frame()
  }
}
```

A list of columns is passed as a json list:

```json
{
  "instruction": "import",
  "file_name": "my_file.csv",
  "cols": ["mpg", "wt"]
}
```

and is converted to a vector (unlist) in the code.  If *cols* does not exist in the json file then `instr$cols` evaluates to `NULL`, as does `unlist(instr$cols)` and the parameter `col_select = NULL` is carried through to `readr::read_csv` or `haven::read_sas`.
