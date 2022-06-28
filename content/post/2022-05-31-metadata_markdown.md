+++
author = "Harvey"
title = "Adding and Retrieving metadata in RMarkdown Documents"
date = "2022-05-31"
description = "Include abitrary metadata in RMarkdown documents"
tags = ["R", "RMarkdown"]
codeMaxLines = 100
draft = false
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

## Searching metadata

Once metadata are added to a series of documents, the metadata become searchable using the `yaml_front_matter` function.  By way of example, the functions below build and then search 1000 documents containing dummy data.  The time taken to search through 1000 documents on a 4-core laptop was 920 ms.

```r
## create Rmd with yaml
create_rmd <- function(ref, name, folder) {

  x <- glue::glue("
---
title: {name}
author: Harvey
short_title: {stringi::stri_rand_strings(1, 20)}
reference: {ref}
output: html_document
params:
    name: {name}
meta_list:
    meta: {ref}
    text: {stringi::stri_rand_strings(1, 20)}
---

### {name}

reference: `r rmarkdown::metadata$reference`

{paste0(stringi::stri_rand_lipsum(5), collapse = '\n\n')}

"
  )

  ## write Rmd file
  con <- file(file.path("./docs", paste0("file_", name, ".Rmd")))
  writeLines(x, con)
  close(con)
}

## build a random set of documents
build_docs <- function(n=10, folder) {
  for (i in seq(n)) {
    create_rmd(ref=i, name=paste("Document", i), folder=folder)
  }
}

## search documents and return matches
search_docs <- function(parameter, search_string, folder) {
  files <- list.files(folder, pattern = "*.Rmd", full.names = TRUE, recursive = TRUE)
  found <- c()
  for (f in files) {
    front_matter <- rmarkdown::yaml_front_matter(f)
    if (grepl(search_string, front_matter[[parameter]])) found <- append(found, f)
  }
  return(found)
}


## create files
build_docs(n=1000, folder="./docs")

## search files
microbenchmark::microbenchmark(
  match_docs <- search_docs(parameter = "short_title",
                            search_string = "ae",
                            folder = "./docs/"),
  times = 5)

```

When run, the search identified 5 documents that matched.

