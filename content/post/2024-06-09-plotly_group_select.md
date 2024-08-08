+++
author = "Harvey"
title = "Selectable Groups in Plotly"
date = "2024-06-09"
description = "How to build a series of selectable groups in Plotly"
tags = ["R", "shiny"]
codeMaxLines = 15
draft = true
+++

This is a short tutorial on building a plotly stripchart with a dropdown to select a dataset.  It covers a few points such as how to handle stripcharts with jitter in plotly, building the plotly menu programatically and how to format a dataset to work with plotly dropdowns.  
Of course, the same effect can be accomplished with a selectizeInput in shiny but this method does not rely on the use of a server [^1] and can be embedded in an HTML report.

## Generating Some Dummy Data
First, we build some dummy data for the plots.

```r
## build a data frame for demo
df <- data.frame(
  id = rep(seq(30), 3),
  group_name = rep(rep(c("A", "B", "C"), each = 10), 3),
  time = rep(paste("Time", seq(3)), each = 30),
  value = lapply(seq(9), function(x) runif(min=x, max=x+1, n=10)) |> unlist()
) |> 
  dplyr::mutate(group_name = as.factor(group_name))
```

|  id | group_name |  time  |  value |
|-----|------------|--------|--------|
|  1  |         A | Time 1 | 1.809334 |
|  2  |         A | Time 1 | 1.021089 |
|  3  |         A | Time 1 | 1.332087 |
|  4  |         A | Time 1 | 1.681937 |
|  5  |         A | Time 1 | 1.623123 |
|  6  |         A | Time 1 | 1.594674 |
| ...  | ... | ... | ... |

## Programatically Building Plotly Dropdown
Changing a dataset interactively in plotly is not as straightforward as a shiny approach of linking a selectizeInput to a parameter.  Plotly's approach requires passing all data to the plot and toggling visibility.  The `updatemenus` parameter in `plotly::layout()` is used to build menus (see https://plotly.com/r/dropdowns/).  
The examples on the plotly site show how to restyle a graph, passing a series of parameters through `updatemenus`.  `updatemenus` itself is a list and can therefore be built programatically.  The code below shows how we can build the UI and logic required to select one of the three values of `time` in our dataset.

```r
## identify the unique values in the time column
time_vals <- df$time |> 
  unique() |> 
  sort()

## count the number of values for each time
v_time_group <- df |>
  dplyr::select(time, group_name) |>
  dplyr::distinct() |>
  dplyr::arrange(time) |>
  dplyr::group_by(time) |>
  dplyr::summarise(count = dplyr::n()) |>
  dplyr::pull(count)

## build a set of vectors to send to updatemenus.
## each member is a set of TRUE/FALSE values denoting visibility.
TF_time_vals <- lapply(seq(length(v_time_group)), function(i) {
  if (i == 1) {
    values_false_start <- 0
  } else {
    values_false_start <- sum(v_time_group[1:(i-1)])
  }

  if (i == length(v_time_group)) {
    values_false_end <- 0
  } else {
    values_false_end <- sum(v_time_group[(i+1):length(v_time_group)])
  }
  c(rep(FALSE, values_false_start), rep(TRUE, v_time_group[i]), rep(FALSE, values_false_end))
})
```

```r
### OUTPUTS for updatemenus
time_vals
[1] "Time 1" "Time 2" "Time 3"

TF_time_vals
[[1]]
[1]  TRUE  TRUE  TRUE FALSE FALSE FALSE FALSE FALSE FALSE

[[2]]
[1] FALSE FALSE FALSE  TRUE  TRUE  TRUE FALSE FALSE FALSE

[[3]]
[1] FALSE FALSE FALSE FALSE FALSE FALSE  TRUE  TRUE  TRUE
```

The `updatemenus` list is built using the code below.  Essentially it is a list of a label, for each time point, and vector of boolean values denoting visibility, for each group at each time point.  It should be noted that each vector for visibility has nine values denoting the three groups (A, B and C) at the three time points (Time 1, Time 2 and Time 3).  In the first, `TF_time_vals[[1]]`, we specify that all three groups are visible for Time 1 but not Time 2 or Time 3.

```r
update_menus_buttons <- lapply(seq_along(time_vals), function(i) {
  list(method = "update",
       args = list(list(visible = TF_time_vals[[i]])),
       label = time_vals[i]
  )
})
```


## Build the Chart
If we build our plot, passing all data at once, the dropdown menu logic fails.  This is because plotly expects a series of separate traces for the dropdown menu.

Therefore this code will not work:

```r
p <- plotly::plot_ly(
  data = df, x = ~jitter(as.numeric(group_name)), y = ~value, type = "scatter", mode = "markers", color = ~group_name
)
```

But this will:
```r
## start with an empty plotly object
p <- plotly::plot_ly(data = df)

for (t in time_vals) {
  d <- df |> dplyr::filter(time == t)
  p <- p |>
    plotly::add_trace(data = d,
                      x = ~jitter(as.numeric(group_name)),
                      y = ~value,
                      type = 'scatter', mode = 'markers',
                      color = ~group_name, visible = t==time_vals[1])
}
```

Since we'll be selecting by time, we neede to create a separate trace, one for each time.  A small amount of jitter is added to the x values.  This is achieved by taking the numeric value of `group_name` (a factor) and applying the base R `jitter()` function.  A caveat to this approach is that the x-axis value are now numeric as opposed to the group name.  Initial visibility is defined as `t==time_vals[1]`, which will take the first value in the `time_vals` vector, namely `Time 1`.

Finally, we add some axis labels and dropdown, and return the plot.

```r
p <- p |> plotly::layout(
  xaxis = list(title = "Group", showticklabels = FALSE, showgrid = FALSE),
  yaxis = list(title = "Values"))

p <- p |> plotly::layout(
  updatemenus = list(
    list(x = 0.1, y = 1.1, buttons = update_menus_buttons)
  )
)

p
```


[^1]: shiny apps can now be built server-free using webR.
