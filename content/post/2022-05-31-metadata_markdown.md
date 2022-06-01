+++
author = "Harvey"
title = "Adding and Retriving metadata in RMarkdown Documents"
date = "2022-05-31"
description = "Include abitrary metadata in RMarkdown documents"
tags = ["R", "RMarkdown"]
codeMaxLines = 30
draft = true
+++

Metadata can be included in the yaml header of an RMarkdown document.  The yaml can store metadata in the `params` parameter or as individual yaml parameters.  For example, the RMarkdown file below adds some metadata parameters to the header: short_title, reference and meta_list.  The `rmarkdown` function `rmarkdown::metadata` can be used to access the yaml parameters.

```markdown
---
title: "document title"
author: "author name"
short_title: "short"
reference: 1
output: "html_document"
params:
    name: "my name"
meta_list:
    meta: "meta 1"
---

test doc

`r rmarkdown::metadata$short_title`

`r params$name`

`r rmarkdown::metadata$meta_list$meta`

```
{{< figure src="/images/post-images/2022-05-31-metadata/metadata_01.png" >}}

In addition, the parameters can be accessed using the `rmarkdown::yaml_front_matter()` function.

```r
rmarkdown::yaml_front_matter("document.Rmd")

$title
[1] "document title"

$author
[1] "author name"

$short_title
[1] "short"

$reference
[1] 1

$output
[1] "html_document"

$params
$params$name
[1] "my name"


$meta_list
$meta_list$meta
[1] "meta 1"
```

