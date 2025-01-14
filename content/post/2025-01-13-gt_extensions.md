
+++
author = "Harvey"
title = "Extending {gt}"
date = "2025-01-13"
description = "Adding Additional Functionality to the gt Package"
tags = ["R"]
codeMaxLines = 15
draft = true
+++

## Introduction

The {gt} package is a great R package for building tables and the outputs that can be generated, particularly in HTML, are stunning.  The {gtExtras} pacakge, from Tom Mock, extends the functionality of {gt}.  In developing a Quarto report, I found that I needed some tabular outputs that were not present in {gt}.  Taking inspiration from {gtExtras}, here's some code and explanation, on adding additional functions to {gt}.

## gtIndex

The `gt_index()` function is a useful function lifted from the {gtExtras} package.  It returns the underlying data of a column and is used extensively in gtExtras functions.  Rather than rely on a dependence to {gtExtras} I've extracted the function, included here as `.gtindex()`.

```r
#' gtindex taken from gtExtras package
#'
.gtindex <- function(gt_object, column, as_vector = TRUE) {
  stopifnot("'gt_object' must be a 'gt_tbl', have you accidentally passed raw data?" = "gt_tbl" %in% class(gt_object))
  stopifnot("'as_vector' must be a TRUE or FALSE" = is.logical(as_vector))

  if (length(gt_object[["_row_groups"]]) >= 1) {
    # if the data is grouped you need to identify the group column
    # and arrange by that column. I convert to a factor so that the
    # columns don't default to arrange by other defaults
    #  (ie alphabetical or numerical)
    gt_row_grps <- gt_object[["_row_groups"]]

    grp_vec_ord <- gt_object[["_stub_df"]] %>%
      dplyr::mutate(group_id = factor(group_id, levels = gt_row_grps)) %>%
      dplyr::arrange(group_id) %>%
      dplyr::pull(rownum_i)

    df_ordered <- gt_object[["_data"]] %>%
      dplyr::slice(grp_vec_ord)
  } else {
    # if the data is not grouped, then it will just "work"
    df_ordered <- gt_object[["_data"]]
  }

  # return as vector or tibble in correct, gt-indexed ordered
  if (isTRUE(as_vector)) {
    df_ordered %>%
      dplyr::pull({{ column }})
  } else {
    df_ordered
  }
}
```

## gt_badge()

this first function simply replaces a column with colored badges.  {gtExtras} includes a `gt_badge()` function itself but 

 - For some reason it's missing from the pkgdown site so I did not know it existed.
 - I required additional functionality - for example, the ability to work with missing data.
 - It was a good first example to sink my teeth into.

 The function is shown below followed by some explanatory text.

```r
#' Add a badge based on values
#'
#' This function differs from gtExtras::gt_badge() in that it accounts for values
#'     missing in the color palette and missing in the data.  Those missing in the
#'     color palette are displayed as a white badge with grey border and those missing
#'     in the data are represented as an empty white badge.
#'
#' @param gt_object An existing gt object
#' @param column The column to convert to dots
#' @param palette Named vector of values and colors. 
#' 
#' @export
#'
gt_badge <- function(gt_object, column, palette = c()) {
  stopifnot("Table must be of class 'gt_tbl'" = "gt_tbl" %in% class(gt_object))

  cell_contents <- .gtindex(gt_object, {{ column }})

  gt::text_transform(
    gt_object,
    locations = gt::cells_body(columns = {{ column }}),
    fn = function(x) {
      purrr::map(cell_contents, function(y) {
        if (is.na(y)) {
          '<span class = "gtbadge gtbadge-empty">none</span>'
        } else if (y %in% names(palette)) {
          glue::glue('<span class = "gtbadge" style = "background-color: {palette[y]};">{y}</span>')
        } else {
          glue::glue('<span class = "gtbadge gtbadge-clear">{y}</span>')
        }
      })
    }
  ) %>%
    gt::opt_css(
      css = "
        .gtbadge {
          display: inline-block;
          color: #ffffff;
          min-width: 30px;
          padding: 2px 4px;
          text-align: center;
          border-radius: 7px;
          font-size: .8em;
        }
        .gtbadge-empty {
          background-color: #ffffff;
          border: 1px solid #dddddd;
        }
        .gtbadge-clear {
          background-color: #ffffff;
          color: #999999;
          border: 1px solid #dddddd;
        }
      "
    )
}
```

First we capture the contents of `column` by using the `.gtindex()` function:  
`cell_contents <- .gtindex(gt_object, {{ column }})` - return the contents of column `column` as a vector (this function is taken from {gtExtras}).

Next, `gt::text_transform()` is used to replace the data in `column` with new values returned by a function.  `gt::text_transform()` is a powerful function that takes three arguments: a gt table, locations for transformation (in this case a column identifier) and a function that returns a character vector the same length as the column entries.  Here, our function iterates over `cell_contents`, the vector of data in `column` and returns a badge.  Badges are colored according to a palette passed to `gt_badge()`, accounting for values missing in the palette as well as empty values.

Finally, the output from `gt::text_transform()` is formatted by declaring the column as markdown and adding css classes.

#### Example Output

```r
df <- data.frame(ref = seq(1:5), data = c("badge_1", "badge_2", "badge_1", NA, "badge_3"))
df |> gt::gt() |> gt_badge(data, palette = c(badge_1 = "#990000", badge_2 = "#009900"))
```

{{< figure src="/images/post-images/2025-01-13-gt_extensions/gt_badge_01.png" >}}

## gt_alert()

```r
This is another simple example.  Here an icon is returned in a column based on TRUE/FALSE values.

#' Replace a logical column with an alert indicator
#'
#' Replace TRUE values with an empty circle and any other values with a red exclamation
#'     mark in a circle.  Setting `invert`=TRUE reverses this behavior.
#'
#' @param gt_object An existing gt object
#' @param column The column to convert to dots
#' @param invert If TRUE then invert the response so that TRUE = alert.  Default is FALSE
#'
#' @export
#'
gt_alert <- function(gt_object, column, invert = FALSE) {
  stopifnot("Table must be of class 'gt_tbl'" = "gt_tbl" %in% class(gt_object))

  cell_contents <- .gtindex(gt_object, {{ column }})

  gt::text_transform(
    gt_object,
    locations = gt::cells_body(columns = {{ column }}),
    fn = function(x) {
      purrr::map(cell_contents, function(y) {
        true_val <- isTRUE(y)
        if (invert == TRUE) {
          true_val <- !true_val
        }
        if (true_val) {
          fontawesome::fa("circle", fill = "#cccccc")
        } else {
          fontawesome::fa("circle-exclamation", fill = "#990000")
        }
      })
    }
  ) %>%
    gt::fmt_markdown(columns = {{ column }})
}
```

#### Example Output

```r
df <- data.frame(ref = seq(1:5), data = c(TRUE, FALSE, NA, TRUE, FALSE))
df |> gt::gt() |> gt_alert(data, invert = TRUE)
```

{{< figure src="/images/post-images/2025-01-13-gt_extensions/gt_alert_01.png" >}}

## gt_dots()

This function displays a vector of values as colored dots.  It's useful for groups containing multiple categorical values.  Column data may be input through a list column, eg list(c("A", "B", "C")) or character-separated, eg "A,B,C".  `gt_dots()` also introduces the use of tooltips.  Code explanation and an example follow below.

```r
#' Replace a column with a series of colored dot rows
#'
#' @param gt_object An existing gt object
#' @param column The column to convert to dots
#' @param items Vector of values for dots.  This represents all of the possible dots in
#'     order
#' @param sep Optional separation character.  If NULL (default) then it is assumed that
#'     column `column` is a list column containing vectors where the member of each vector
#'     can be a value in `items`.  For example, if `items` is c("A", "B", "C") then `column`
#'     could contain data such as "A" or c("A", "B").  If a `sep` is a character then column
#'     `column` should be a character vector with values separated by `sep`.  For example, if
#'     `items` is c("A", "B", "C") and `sep` is ";" then `column` could contain data such as
#'     "A" or "A;B".
#' @param tooltip If TRUE then add a tooltip indicating the active values
#'    
#' @return gt table
#'
#' @export
#'
gt_dots <- function(gt_object, column, items = c(), sep = NULL, tooltip = FALSE) {
  stopifnot("Table must be of class 'gt_tbl'" = "gt_tbl" %in% class(gt_object))
  
  cell_contents <- .gtindex(gt_object, {{ column }})
  
  pal <- colorRampPalette(c("#89CFF1", "#003A6B"))
  cols <- pal(length(items))
  l_dots <- lapply(seq_along(items), function(i) {
    fontawesome::fa("fas fa-circle", fill = cols[i], margin_left = '.05em', margin_right = '.05em')
  }) |>
    setNames(items)
  blank <- fontawesome::fa("far fa-circle", fill = "#cccccc", margin_left = '.05em', margin_right = '.05em')
  
  col_name <- rlang::quo_name(rlang::quo({{column}}))
  width_val <- length(items) * 1.1 + 1
  colwidth <- as.formula(paste0(col_name, " ~ '", width_val, "em'"))
  
  gt::text_transform(
    gt_object,
    locations = gt::cells_body(columns = {{ column }}),
    fn = function(x) {
      lapply(cell_contents, function(y) {
        
        # split to create a vector
        if (!is.null(sep)) {
          y <- unlist(strsplit(y, sep, fixed = TRUE))
        }
        
        # find matches
        dot_matches <- match(y, items)
        dots <- rep(blank, times = length(items))
        if (!is.na(dot_matches[1])) {
          for (i in dot_matches) {
            dots[i] <- l_dots[[i]]
          }
        }
        
        output_dots <- paste(dots, collapse = "")
        
        if (tooltip == TRUE) {
          if (!is.na(dot_matches[1])) {
            tooltip_text <- paste(items[dot_matches], collapse = ", ")
          } else {
            tooltip_text <- NA
          }
          
          glue::glue(
            "<div data-bs-toggle='tooltip' data-bs-placement='right' data-bs-title=\"{tooltip_text}\">{output_dots}</div>"
          )
          
        } else {
          output_dots
        }
      })
    }
  ) %>%
    gt::fmt_markdown(columns = {{ column }}) %>%
    gt::cols_width(colwidth)
}
```


At the beginning of the function we build a named vector of colors.  We've hardcoded shades of blue but could easily pass the two color-extremes as parameters to `gt_dots()`.  The output is `l_dots`, a named list of colored icons, and `blank`, an empty icon.

```r
  pal <- colorRampPalette(c("#89CFF1", "#003A6B"))
  cols <- pal(length(items))
  l_dots <- lapply(seq_along(items), function(i) {
    fontawesome::fa("fas fa-circle", fill = cols[i], margin_left = '.05em', margin_right = '.05em')
  }) |>
    setNames(items)
  blank <- fontawesome::fa("far fa-circle", fill = "#cccccc", margin_left = '.05em', margin_right = '.05em')
```

The next part of the code builds a formula defining the column width.  The column width is defined by the number of dots.  This keeps all the dots on a single line.

```r
  col_name <- rlang::quo_name(rlang::quo({{column}}))
  width_val <- length(items) * 1.1 + 1
  colwidth <- as.formula(paste0(col_name, " ~ '", width_val, "em'"))
```

Finally we run the `gt::text_transform()` function, looping over each vector of data.  The output is a series of colored dots, corresponding to matches against a vector.  If requested, a tootlip is added for each column cell.

{{% notice note "Tooltips in Quarto" %}}
Note, to use tooltips in a quarto HTML document, Bootstrap tooltips need to first be activated.  This can be done by adding the following in the document's yaml header:

```yaml
include-after-body:
- text: "<script>\n  const tooltipTriggerList = document.querySelectorAll('[data-bs-toggle=\"tooltip\"]')\n
    \ const tooltipList = [...tooltipTriggerList].map(tooltipTriggerEl => new bootstrap.Tooltip(tooltipTriggerEl, {html: true}))\n</script>\n"
```
{{% /notice %}}

#### Example Output

```r
df <- data.frame(ref = seq(1:5), data = c("p1,p2", "p1", NA, "p3", "p2,p3"))
df |> gt::gt() |> gt_dots(data, items = c("p1", "p2", "p3", "p4"), sep = ",", tooltip = TRUE)
```

{{< figure src="/images/post-images/2025-01-13-gt_extensions/gt_dots_01.png" >}}
