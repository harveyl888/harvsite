+++
author = "Harvey"
title = "visNetwork Continuous Colors"
date = "2019-03-15"
description = "Building a visNetwork graph with a continuous color-coded fill"
tags = ["R"]
+++


visNetwork is a fantastic library used to make interactive network graphs in R.  It has many features to style nodes and edges but lacks a way to fill nodes using a continuous variable.  ThIs can readily be achieved by using findInterval to map a continuous variable to the closest color on a color scale.  In the example below a random data set is generated with a probability between zero a one attached to each node.  Color is then assigned from a palette of 101 shades of blue.  The resultant graph illustrates the effect of coloring using a continuous variable - higher probabilities are darker (label = node number and probability).

```r
## visNetwork continuous color

library(visNetwork)
library(igraph)
library(dplyr)
library(RColorBrewer)

## define number of nodes, edges and colors

n_nodes <- 100
n_edges <- 200
n_cols <- 101
pal <- colorRampPalette(brewer.pal(9, 'Blues'))(n_cols)

## create graph from random edges
## p is a variable for color coding from 0 to 1

df.nodes <- data.frame(id = seq(n_nodes), p = runif(seq(n_nodes))) %>%
  mutate(label = sprintf('n %i (%0.2f)', id, p)) %>%
  mutate(c_ref = findInterval(p, seq(from = 0, to = 1, length.out = n_cols))) %>%
  mutate(color = pal[c_ref])
  
df.edges <- data.frame(from = sample(seq(n_nodes), n_edges, replace = TRUE), to = sample(seq(n_nodes), n_edges, replace = TRUE))

visNetwork(nodes = df.nodes ,edges = df.edges) %>%
  visIgraphLayout()
```
{{< figure src="/images/post-images/2019-03-15-visnetwork_continuous/visnetwork.png" >}}

