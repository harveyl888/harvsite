+++
author = "Harvey"
title = "{distill} Comments With Replies"
date = "2022-03-05"
description = "Including a comments (plus replies) section in {distill} pages"
tags = ["R", "javascript", "rsconnect"]
draft = true
+++

This post expands upon the post on [{distill} Comments](/post/2022-01-11-distill_comments/).  It includes a method to reply to comments and store comments and replies in an hierarchical manner.

In the previous post I covered how we could use RStudio Connect to manage commenting on a static blog.  Here we extend it, adding a way to reply to comments and store comments plus replies in a hiersrchical data structure.

The concept is essentially the same as the earlier version: a {distill} blog is connected to a {pins} data source via plumber.  Here, however, the data source is a data.tree as opposed to a data frame.  [data.tree](http://gluc.github.io/data.tree/) is an R package that manages hierarchical data and tree structures.  Page comments with replies lends itself nicely to a hierarchical data structure where each node is a comment or reply to a comment.  The pinned data.tree holds the comments and replies which can be added or retrieved through the API.  Comments are retrieved through javascript functions in the distill blog.  The blog, pin board and plumber API all sit on the same RStudio Connect instance.

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_01.png" >}}