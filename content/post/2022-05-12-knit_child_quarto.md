+++
author = "Harvey"
title = "knitr::knit_child with Quarto"
date = "2022-05-12"
description = "knitr::knit_child with Quarto"
tags = ["RMarkdown"]
codeMaxLines = 30
+++

[Quarto](https://quarto.org/) is the new publishing system from RStudio based on Pandoc.  It's a powerful tool for publising from R, Python and Julia, working in a very similar fashion to RMarkdown.  
There are several great blog posts already highlighting [features of Quarto](https://www.apreshill.com/blog/2022-04-we-dont-talk-about-quarto/) and [thoughts on changing from RMarkdown to Quarto](https://www.njtierney.com/post/2022/04/11/rmd-to-qmd/).

Recently, I've been building up dynamic documents using child documents.  The aim is to identify the parts of a document (report) required and compile them into a main report as child documents.  I've also started to use Quarto, but can Quarto build a document from a selection of child docs?  The answer is 'yes'.

## RMarkdown Version

knitr Child documents are a great way to manage long reports.  They provide a method to break a document into sections and knit the sections together upon rendering.  Below we have a main Rmd document and a child Rmd document.  When rendered, the child document is incorporated into the main document.

### Main Document

~~~r
---
title: "Child docs (RMarkdown)"
output: html_document
---

Test quarto document using child docs - RMarkdown verion

```{r, include=FALSE}
library(knitr)
```

```{r, results='asis', echo=FALSE}
a <- knitr::knit_child('child_doc_01.Rmd', quiet=T)
cat(a, sep='\n')
```
~~~

### Child Document 1

```r
---
tag: "my child 01"
title: "child doc 01"
---

## Child Doc 1

Child document #1 - this is an RMarkdown child document
```

### Output

{{< figure src="/images/post-images/2022-05-12-quarto/quarto_01.png" >}}

## Quarto Version

Quarto child documents can be knitted in the same way as RMarkdown documents, using knitr::knit_child.  In fact Quarto and RMarkdown child documents can be mixed as shown below.

### Main Document

```r
---
title: "Child docs"
format: html
execute: 
  echo: false
self-contained: true
---

Quarto child documents are compatible with knitr::knit_child

~~~{r}
#| include: false
library(knitr)

```

```{r, results='asis'}
a <- knitr::knit_child('child_doc_01.Rmd', quiet=T)
cat(a, sep="\n")
```

```{r, results='asis'}
a <- knitr::knit_child('child_doc_02.qmd', quiet=T)
cat(a, sep="\n")
```
~~~

### Child Document 2

---
title: "Untitled"
format: html
---

## Child Doc 2

~~~r
Child document #2 - this is a quarto child document running {ojs} cell

```{r}
ojs_define(data = mtcars)

```

```{ojs}
Plot.plot({
  marks: [
    Plot.dot(transpose(data), 
      {x: "mpg", y: "hp", stroke: "cyl"}
    )
  ]}
)


```
~~~

### Output

{{< figure src="/images/post-images/2022-05-12-quarto/quarto_02.png" >}}
