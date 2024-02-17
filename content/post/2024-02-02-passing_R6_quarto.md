+++
author = "Harvey"
title = "Passing and R6 Class to Quarto"
date = "2024-02-02"
description = "how to pass an R6 class to Quarto as a parameter"
tags = ["R", "Quarto"]
codeMaxLines = 15
draft = false
+++

Quarto has many improvements over RMarkdown, plus a couple of limitations, one being that you cannot pass R objects as parameters to a Quarto document in the way the RMarkdown would accept them.  Quarto receives parameters serialized as yaml, so only text, numeric, boolean (received as *TRUE="yes"* and *FALSE="no"*) and lists can be passed.  There are instances when you may wish to pass a complex object, for example an R6 class instance, as a paramater for Quarto.  One way to do this is to first serialize the object and then pass it as a serialized JSON object.

### Preparing data

The code below simply creates a data frame and a list.  The data frame contains 1000 rows and 1000 columns and the list contains 1000 members.

```r
n_row <- 1000
n_col <- 1000

d <- as.data.frame(matrix(rnorm(n_row * n_col), n_row, n_col))
l <- as.list(rnorm(n_row))
```

In order to explore passing data vs. passing an R6 class we'll create a simple R6 class:

```r
R <- R6::R6Class("R",
  public = list(
    data = NULL,
    list = NULL,
    initialize = function(data = NULL, list = NULL) {
      self$data = data
      self$list = list
    }
  )
)
```

This class simply holds the data frame and list and returns them.  We can create an instance of the class with the data defined above as follows:

```r
r <- R$new(data = d, list = l)
```

## Quarto Doc 1 - sending data frame and list as parameters

The first Quarto document, `quarto_01.qmd` accepts two parameters - a data frame and a list.

~~~quarto
---
title: "data test 01"
format: html
engine: knitr
params:
  mydata: NULL
  mylist: NULL
---

Imported two items: tibble and list.  
Number of rows in tibble: `r nrow(params$mydata)`  
Number of items in list: `r length(params$mylist)`
~~~

Rendering the Quarto document produces the following output.  It should be noted that when rendered, the data frame is passed as a list since Quarto does not have a concept of data frames and therefore the term `nrow(params$mydata)` fails to report the number of rows.

```r
quarto::quarto_render("quarto_01.qmd", execute_params = list(mydata = d, mylist = l))
```

{{< figure src="/images/post-images/2024-02-02-passing-R6-quarto/R6_quarto_01.png" >}}

In order to pass the data frame we'll need to serialize it to a JSON representation and unserialize it once passed.  We can do this using {jsonlite} as follows:

~~~quarto
---
title: "data test 01A"
format: html
engine: knitr
params:
  mydata: NULL
  mylist: NULL
---

```{r}
#| echo: false
df <- jsonlite::fromJSON(params$mydata)
```

Imported two items: data frame and list.  
Number of rows in data frame: `r nrow(df)`  
Number of items in list: `r length(params$mylist)`
~~~

Now when rendered, we get the following output:

```r
quarto::quarto_render("quarto_01A.qmd", execute_params = list(mydata = jsonlite::toJSON(d), mylist = l))
```

{{< figure src="/images/post-images/2024-02-02-passing-R6-quarto/R6_quarto_01A.png" >}}

## Quarto Doc 2 - sending the R6 class instance as a parameter

The final Quarto document, `quarto_02.qmd` accepts just a single parameter for the R6 class instance.  In this case we serialize the R6 class and then pass it using `jsonlite::JSONserialize()`.  When received by the Quarto document we simply reverse the procedure.  The class itself has to be serialized first as using `jsonlite::JSONserialize()` on the class instance, `r` results in passing the class structure as a parameter and not the class instance.

~~~quarto
---
title: "data test 02"
format: html
engine: knitr
params:
  my_serialized_class: NULL
---

```{r}
#| echo: false
r_class <- unserialize(jsonlite::unserializeJSON(params$my_serialized_class))
```

Imported one items: R6 class.  
Number of rows in data frame: `r nrow(r_class$data)`  
Number of items in list: `r length(r_class$list)`
~~~

Rendering the document leads to the expected output.

```r
param_r6 <- jsonlite::serializeJSON(serialize(r, connection = NULL))
quarto::quarto_render("quarto_02.qmd", execute_params = list(my_serialized_class = param_r6))
```

{{< figure src="/images/post-images/2024-02-02-passing-R6-quarto/R6_quarto_02.png" >}}


## Performance

Times to render (including serializing and deserializing) are shown below.  Surprisingly passing the R6 class was faster than than the data frame.

| description | time (secs) |
|--|--|
| data frame and list, no serialization |  9.0 |
| data frame and list, data frame serialized | 13.9 |
| R6 class, serialized | 5.4 |

The effect is even more pronounced as the quantity of data increases.  When *n_row* is increased 10-fold to 10000, the times become:

| description | time (secs) |
|--|--|
| data frame and list, no serialization |  43.5 |
| data frame and list, data frame serialized | 80.3 |
| R6 class, serialized | 14.3 |

## Conclusion

Complex objects, such as R6 classes, can be used in parameterized Quarto documents if properly serialized.  Passing a serialized R6 class is more efficient than passing an equivalent data frame / list combination.
