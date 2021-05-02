+++
author = "Harvey"
title = "CAS from pdf"
date = "2019-12-24"
description = "CAS from pdf"
tags = ["R", "Chemistry"]
+++

It's relatively easy to extract CAS numbers from a PDF.  I've used this to scrape chemical catalogs, subsequently using the CAS numbers to look up additional information through PubChem.  CAS numbers are easier to scrape than most other chemical information, such as name, formula or catalog number, as they follow a specific format.

It can be accomplished using a couple of lines of code in R (or collapsed into a single line if you prefer).  The code below returns all CAS numbers from a file, *f*.

```r
pdf_to_cas <- function(f) {
  txt <- pdftools::pdf_text(f)
  cas_numbers <- trimws(unlist(stringr::str_extract_all(txt, '[0-9]+[\u2011|-][0-9]{2}[\u2011|-][0-9](?![0-9])')))
  return(cas_numbers)
}
```

The regular expression `[0-9]+[\u2011|-][0-9]{2}[\u2011|-][0-9](?![0-9])` can be broken down as follows:

`[0-9]+`          one or more digits  
`[\u2011|-]`      a dash or an unbreaking hyphen  
`[0-9]{2}-`       exactly two digits  
`[\u2011|-]`      a dash or an unbreaking hyphen  
`[0-9](?![0-9])`  a single digit followed by anything other than a digit using a negative lookahead 
