+++
author = "Harvey"
title = "Toggling Error Bars in Plotly"
date = "2024-02-15"
description = "adding buttons to turn error bars on and off"
tags = ["R", "Shiny"]
codeMaxLines = 15
draft = true
+++

Plotly charts can be enhanced with custom controls.  One use of controls is to update an existing chart.  This post will show how to add a couple of buttons to a chart, toggling error bars on and off.  It should be noted that [coloring by group](https://github.com/plotly/plotly.R/issues/762) affects the order (and hence validity) of error bars in a Plotly plot so a [workaround](/post/2017-10-26-plotly_error_bars/) has to be employed.  

### Generate Some Demo Data

```r
set.seed(12345)
d <- data.frame(
 time = rep(0:9, 10),
  group = sample(paste0("Group_", seq(5)), size = 1000, replace = TRUE),
  value = rnorm(n = 1000) + 1
) |>
  dplyr::group_by(time, group) |>
  dplyr::summarise(mean = mean(value), se = sd(value) / sqrt(dplyr::n())) |>
  dplyr::ungroup()
```

### Initializing Chart and Adding Traces

We'll build up the chart from the demo data by first initializing an empty plot as follows:

```r
p <- plotly::plot_ly(type = 'scatter', mode = 'lines+markers')
```

The next step is to add a series of traces, one for each group, without error bars:

```r
for (x in unique(d$group)) {
  p <- p |>
    plotly::add_trace(data = d |> dplyr::filter(group == x), visible = TRUE,
                      x = ~time, y = ~mean, type = 'scatter', mode = 'lines+markers', color = ~group)
} 
```

and then a second series of traces, this time with error bars but with the parameter `visible=FALSE` set to ensure that they are not visible when the chart is initially drawn:

```r
for (x in unique(d$group)) {
  p <- p |>
    plotly::add_trace(data = d |> dplyr::filter(group == x), visible = FALSE,
                      x = ~time, y = ~mean, type = 'scatter', mode = 'lines+markers', color = ~group,
                      error_y = ~list(array = se, color = group))
}
```

Since we have 5 groups, our plotly chart now contains 5 traces without error bars and 5 traces with error bars.  It actually contains one additional empty trace corresponding to the initial `plotly::plot_ly()` call when the empty chart was built.  This can be deduced using `plotly::plotly_build()` to observe the list object sent to plotly for plotting.  At this stage the following code reveals a total of 11 traces:

```r
p_obj <- plotly::plotly_build(p)
print(length(p_obj$x$data))
```

### Building the Menu Buttons 

Plotly does not have the option of a toggle switch or toggle button so we'll add two buttons - one to plot without error bars and one to plot with error bars.  The control works by changing the visible status so that only a subset of plots are visible.  As mentioned above, we have 5 traces without error bars and 5 with.  We also have the empty trace (the first trace), so in order to see just traces without error bars we'll want to show only traces 2-6 and in order to see just traces with error bars we'll want to show only traces 7-11.  Here is the code to build the plotly menu buttons:

```r
num_traces <- length(unique(d$group))
menu <- list(
  active = 0,
  type = 'buttons',
  direction = 'right',
  x = 1.2,
  y = 0.1,
  buttons = list(
    list(
      label = 'off',
      method = 'update',
      args = list(list(visible = c(F, rep(c(TRUE, FALSE), each = num_traces))))
    ),
    list(
      label = 'on',
      method = 'update',
      args = list(list(visible = c(F, rep(c(FALSE, TRUE), each = num_traces))))
    )
  )
)

annotation <- list(list(text = "Error Bars", x = 1.18, y = 0.13, xref = "paper", yref = "paper", showarrow = FALSE))
```

### Plotting

Finally, the plot can be created by adding the menu items and annotation to the existing plotly object:

```r
p |>
  plotly::layout(updatemenus = list(menu), annotations = annotation)
```

{{< figure src="/images/post-images/2024-02-15-toggle_plotly_errorbars/errorbars_off.png" >}}

{{< figure src="/images/post-images/2024-02-15-toggle_plotly_errorbars/errorbars_on.png" >}}
