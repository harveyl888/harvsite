+++
author = "Harvey"
title = "Embedding {learnr} in {distill}"
date = "2021-11-30"
description = "How to share a {learnr} lesson within a {distill} website page"
tags = ["R"]
+++

{learnr} is an R package from RStudio which uses R Markdown to create interactive tutorials.  Once completed, the tutorial can be published via shiny server or RStudio Connect.  As a {learnr} markdown file contains its own yaml header it cannot be directly included in a website built by distill.  It can, however, be included if it is hosted (shiny server or RStudio Connect) and embedded using `iframe`

~~~{r}
```{r, layout="l-screen-inset"}
htmltools::tags$iframe(src = "<RSCONNECT URL>", height = "600px", width = "100%", `data-external` = "1")
```
~~~

where *<RSCONNECT URL>* is the url of the {learnr} tutorial.  The use of `data-external` is related to the way that pandoc processes iframe tags (see https://community.rstudio.com/t/insert-raw-html-iframe-into-rmarkdown/94334 and https://pandoc.org/MANUAL.html#option--self-contained).

{{< figure src="/images/post-images/2021-11-30-learnr_distill/learnr_distill.jpg" >}}
