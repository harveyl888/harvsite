+++
author = "Harvey"
title = "chemSymWords"
date = "2017-03-29"
description = "Building words using chemical symbols"
tags = ["R", "Chemistry"]
+++



There seems to be a fad right now of merchandising articles containing words made up of chemical symbols.  Beryllium Erbium (BeEr) is particularly popular and I have to admit I have a T-Shirt claiming "Call me NErDy".  I've put together a small code in R which will check to see if a word can be created purely by element symbols and then output a PNG.  

The algorithm to turn a word into symbols is fairly straightforward and involves a little recursion.  The recursive function is shown below.  It comprises of an outer function (`chemwordRecurse`) and an inner one `chemwordRecurseInternal`.  The output variable (a list) is initialized in the outer function and utilized within the inner one.  This allows a single output to be updated which can be a problem when recursion traverses a tree-like pathway as there could be multiple instances of the output.  `Recall` is a way to recursively call using a placeholder as opposed to specifying the function name.

```r
chemWordRecurse <- function(w, t, sym=elements) {
  out <- list()

  chemWordRecurseInternal <- function(w, t, sym) {
    if (nchar(w) == 0) {
      out[[length(out) + 1]] <<- unlist(t)
    }
    for (i in 1:min(max(nchar(sym)), nchar(w))) {
      find_el <- match(toupper(substring(w, 1, i)),toupper(sym))
      if (!is.na(find_el)) {
        Recall(substring(w, i + 1), c(t, find_el), sym)
      }
    }
  }
  chemWordRecurseInternal(w, t, sym)
  return(out)
}
```

An old stackexchange thread ([http://puzzling.stackexchange.com/questions/5456/the-longest-word-made-from-chemical-symbols](http://puzzling.stackexchange.com/questions/5456/the-longest-word-made-from-chemical-symbols)) challenges the longest word which can be made out of chemical symbols.  At the time of this writing it's Floccinaucinihilipilifications at 30 letters.

{{< figure src="/images/post-images/2017-03-29-chemSymWords/Floccinaucinihilipilifications.png" >}}

The code can be found at [https://github.com/harveyl888/chemSymWords](https://github.com/harveyl888/chemSymWords)