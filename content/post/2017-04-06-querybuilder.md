+++
author = "Harvey"
title = "queryBuilder"
date = "2017-04-06"
description = "An htmlwidget for jQuery QueryBuilder"
tags = ["R", "shiny"]
+++

queryBuilder is an htmlwidget wrapper for [jQuery queryBuilder](http://querybuilder.js.org/), a fantastic jQuery plugin to create queries and filters.  The htmlwidget generates a dplyr string that can be used to filter a data frame within a shiny app.  In addition I'm playing with some experimental features to expand functionality.  The first is a way to filter on up/down trend, something that I've used in metabolomics studies.  The second is a group comparison feature, something that's achieved by switching out the html on the fly - toggling between a textbox and a combobox.  An example of the group comparison filter can be found on [codepen](https://codepen.io/harveyl888/pen/mOejgN) and the htmlwidget is hosted on [github](https://github.com/harveyl888/queryBuilder).

{{< figure src="/images/post-images/2017-04-06-queryBuilder/queryBuilder.png" >}}
