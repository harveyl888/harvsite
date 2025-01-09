
+++
author = "Harvey"
title = "New Year Fun with Quarto"
date = "2025-01-02"
description = "A New Year's balloon pop updated using Quarto"
tags = ["Quarto"]
codeMaxLines = 15
draft = false
+++

## Introduction

We have a New Year tradition of a balloon pop for the kids.  We blow up a number of balloons, each containing a piece of paper with a fun game or activity.  Throughout New Year's Eve we pop a balloon every half hour and the kids enjoy the surprise.  Of course, it's much more fun to build this in Quarto so here we go.  The repo is available at https://github.com/harveyl888/newyear.

## Balloon Pop Quarto

The concept is simple.  Inside each balloon is a QR code instead of an activity.  The QR code points to an entry on a Quarto blog.  Kids pop a balloon, scan the code, and the fun activity pops up.  There's a distinct advantage to this approach over the old paper method - by decoupling the activity from the balloon we can easily change the activity (by updating the blog) without having to destroy the balloon.

## QR Codes

Generating QR codes is fairly straightforward with the {qrcodes} package.  The following Quarto file generates a QR code for each entry in the `posts` folder.  It reads the websites's `site-url` parameter from `_quarto.yml`, construct's each post's URL, generates the QR code and outputs the codes to a page on the site (see example at https://harveyl888.github.io/newyear/QRcodes.html).

````
---
title: QR Codes
---

```{r}
#| label: build_codes
#| echo: false
#| results: asis

quarto_yaml <- yaml::read_yaml("_quarto.yml")
url <- quarto_yaml$website$`site-url`
posts <- list.files("posts")

cat("::: {#fig-QR layout-ncol=3}\n")

for (i in seq_along(posts)) {
  # generate a QR code
  qr <- qrcode::qr_code(file.path(url, "posts", posts[i]))

  # save QR code as svg
  f <- file.path("qr_svg", sprintf("f%02i.svg", i))
  qrcode::generate_svg(qr, filename = f, show = FALSE, size = 200)

  # get title and add as label
  post_yaml <- rmarkdown::yaml_front_matter(file.path("posts", posts[i], "index.qmd"))
  cat(paste0("![", post_yaml$title,  "](", f, ")\n\n"))
}

cat("QR Codes\n\n:::\n")

```
````

## Extensions

One of the great things about Quarto is the ability to write extensions.  I added two simple shortcode extensions for a game of charades and to give a list of tongue-twisters.  Quarto extensions are written in lua but, in this case, the extensions simply take some parameters from the post header and build a button, passing the parameters to a javascript function.  The extensions can be found at https://github.com/harveyl888/newyear/tree/main/_extensions/newyear.

## Conclusion

Overall, a fun, simple idea which only took a few hours to put together from scratch.

{{< figure src="/images/post-images/2025-01-02-new_year_quarto/balloon_pop_01.jpg" width="50%" >}}