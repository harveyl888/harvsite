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

## New Comment Form

New comment form is very similar to the original version.  The function `comment_form_dt` takes `site_id` and `page_id` arguments and returns an HTML form.  `site_id` is a unique identifier for a website and `page_id` is a unique identifier for a page on that site.  
The form captures a comment and optional user name and passes each of these, along with `site_id`, `page_id` and `parent_ref` to a plumber API.  Each comment or reply is given a unique reference number and `parent_ref` is the reference number of the parent.  For page comments `parent_ref` is simply the `page_id` but for replies `parent_ref` is the reference to a comment or a reply.  The plumber API updates a *pinned* data.tree with the new comment.  In fact, a javascript function intercepts the submit button triggering an update of the page comments after adding the new one.  This allows a new comment to be added without having to refresh the page manually.  
In addition, the `comment_form_dt` function adds a div with the id `rtncomments` which is a placeholder to display comments.  
The `comment_form_dt` R function along with the javascript eventListener are shown below.  In the code, *<rsconnect URL>/addcomment_dt* refers to the plumber API endpoint for adding a new comment.  
The javascript function *formsubmit* is essentially the same as the earlier function.

```r
library(htmltools)

comment_form_dt <- function(page_id = 0, site_id = 0) {
  
  comment_html <- paste0('
  <div class="comments">
    <div class="form-container">
      <h3 class="comment-header">Post a Comment</h3>
      <form action="<rsconnect URL>/addcomment_dt" id="my-form">
      
        <div class="form-contents">
          <span class="comment-pic">
            <i class="far fa-user"></i>
          </span>
          
          <div class="form-details">
            <div class="comment-comments">
              <input type="text" id="comment" name="comment" placeholder="Your comment"></textarea>
            </div>
            <div class="comment-user">
              <span class="comment-short">
                <input type="text" id="user_name" name="user_name" placeholder="Your name (optional)" />
              </span>
            </div>
          </div>

          <input type="hidden" name="site_id" value="', site_id, '" />
          <input type="hidden" name="page_id" value="', page_id, '" />
          <input type="hidden" name="parent_ref" value="', page_id, '" />
      
          <span class="button-container">
            <input type="submit" value="Comment">
          </span>
        </div>
      </form>
    </div>
      <div id="rtncomments">
    </div>
  </div>
  ')
  htmltools::HTML(comment_html)
}
```

```js
window.addEventListener("load", function() {
  // add eventlistener to new comment submit button
  document.getElementById("my-form").addEventListener("submit", formsubmit);
});

// Intercept submit button and run fetch code
async function formsubmit(e) {
  
  e.preventDefault();
  
  // get event-handler element
  const form = e.currentTarget;
  
  // get form url
  const url = form.action;
  
  // get form data as json string
  formdata = new FormData(form);
  const plainFormData = Object.fromEntries(formdata.entries());
  const jsonFormData = JSON.stringify(plainFormData);
  
  // send request and capture output
  out = await fetch(url, {
    method: 'POST',
    body: jsonFormData,
    headers: {
      "Content-Type": "application/json",
      "Accept": "application/json"
    }
  })
  .then(response => response.json());
  
  // update comments
  update_comments_dt(plainFormData.page_id, plainFormData.site_id);

};

```

## Existing Comments

Retrieving existing comments introduces a new function to build replies and a reply box for each comment/reply.  The main function takes `site_id` and `page_id` arguments and calls a plumber API which returns comments belonging to the page in json form.  A recursive function then builds comments and any replies, terminating each tree branch with a reply box.  
Here, *<rsconnect URL>/page_comments_dt* refers to the plumber API endpoint for retrieving comments.  The search parameters `site_id` and `page_id` are appended to the url so that we can limit the returning data to a specific page on a specific site.  Since we are using fetch, the webpage and API must live on the same RStudio Connect instance.  

```js
// build and populate comment reply box
function reply_comment_box(page_id, site_id, parent_ref) {
  var out = $('<div/>', {class: 'form-container'}).append([
    $('<h5/>', {class: 'comment-header comment-header-margin-narrow', text: 'Post a reply'}),
    $('<form/>', {action: 'https://rsconnect-prod.dit.eu.novartis.net/content/1200/addcomment_dt', method: 'POST', class: 'reply-form'}).append([
      $('<div/>', {class: 'form-contents'}).append(
        $('<span/>', {class: 'comment-pic'}).append($('<i/>', {class: 'far fa-user'})),
        $('<div/>', {class: 'form-details'}).append(
          $('<div/>', {class: 'comment-comments'}).append(
            $('<input/>', {type: 'text', name: 'comment', placeholder: 'Your reply'})
          ),
          $('<div/>', {class: 'comment-user'}).append(
            $('<span/>', {class: 'comment-short'}).append(
              $('<input/>', {type: 'text', name: 'user_name', placeholder: 'Your name (optional)'})
            )
          )
        ),
        $('<input/>', {type: 'hidden', name: 'site_id', value: site_id}),
        $('<input/>', {type: 'hidden', name: 'page_id', value: page_id}),
        $('<input/>', {type: 'hidden', name: 'parent_ref', value: parent_ref}),
        $('<span/>', {class: 'button-container'}).append(
          $('<input/>', {type: 'submit', value: 'submit'})
        )
      )

    ])
  ])
  return(out)
};


// update comments on the page
function update_comments_dt(page_id, site_id) {

  const url = "<rsconnect URL>/page_comments_dt?"

  fetch(url + new URLSearchParams({
    site: site_id, 
    page: page_id,
  }))
  .then(response => response.json())  
  .then(data => {
    
    // recursive function to print comments
    function comment_recurse(d) {
      if (d.hasOwnProperty('children')) {
        const ul_list_comments = $('<ul/>', {class: 'comment-list'});

        // loop over children (replies) and populate
        $.each(d.children, function(i, x) {
          user_name = x.user_name == "null" ? "anonymous user" : x.user_name;
          style_txt = 'margin-left: 20px;'
          ul_list_comments.append(
            $('<li/>', {class: 'comment-item', style: style_txt}).append([
              $('<div/>', {class: 'comment-top'}).append([
                $('<h3/>', {class: 'comment-name', text: user_name}),
                $('<span/>', {class: 'date-holder'}).append([
                  $('<i/>', {class: 'far fa-clock'}),
                  $('<h3/>', {class: 'comment-date', text: x.date})
                ])
              ]),
              $('<p/>', {class: 'comment-text', text: x.comment}),
              $('<details/>').append([
                $('<summary/>', {class: 'text-reply', text: 'reply'}),
                reply_comment_box(x.page_id, x.site_id, x.ref)
              ]),
              comment_recurse(x)
            ]),

          );
          
        });
        return(ul_list_comments)
      } else {
        return(null)
      }
      
    }
      
    // outer_div - placeholder for comments
    div_outer = $('<div/>').attr('id', 'div_outer');
    
    // add comments if exist
    if (data.children) {
      // add comments count
      div_outer.append('<h3>' + data.children.length + ' Comments</h3>');
      
      // recursively loop through returned comments, building unordered lists
      ul_list_comments = comment_recurse(data);
      
      // add comments to outer div
      div_outer.append(ul_list_comments);
    }
    
    // update comment holder
    $("#rtncomments").html(div_outer);
    
    // add event listener to class
    const reply_forms = document.querySelectorAll('.reply-form');
    reply_forms.forEach(item => item.addEventListener('submit', formsubmit));
    
  })
  .catch((err) => console.log("Can’t access " + url + " response. Blocked by browser?" + err));
  
};
```

## plumber API

As [previously](/post/2022-01-11-distill_comments/), the distill blog pages via a plumber API.  The API contains two endpoints, a POST endpoint, `addcomment_dt` which is used to add a new comment and a GET endpoint, `page_comments_dt` which is used to retrieve comments for a specific page.  The comments are stored in a hierarchical data.tree format which is accessible via the {pins} package.

In the code below, `board_register("rsconnect", server = "<rsconnect URL>, account = "<account id>", key = connectAPIKey)` registers a pin board which holds a pin called `blog_comment_table+dt`.  *<rsconnect URL>* refers to the RStudio Connect URL and, *<account id>* is the account associated with the pin.  An RStudio Connect API key must be defined and exposed as an environment variable (see below).

{{< figure src="/images/post-images/2022-01-11-distill_comments/distill_comments_04.png" width="75%" >}}

`addcomment_dt` adds the comment to a `parent id` which sits under a `page id`, that is, in turn, under a `site id`.  Each comment is given a unique reference id, used as an identifier when comments or replies are added.

`page_comments_dt` retrieves a hierarchy of comments and replies for a specified `site id` and `page id`.  The data.tree obtained is returned as a list.


```r
library(plumber)
library(jsonlite)
library(pins)
library(lubridate)
library(data.tree)
library(stringi)

#* Add a comment to the comment table
#* 
#* @param req request body
#* 
#* @serializer unboxedJSON
#* 
#* @post /addcomment_dt
function(req) {
  
  ## get the message body
  body <- jsonlite::fromJSON(req$postBody)
  
  ## RSConnect API Key
  connectAPIKey <- Sys.getenv("CONNECT_API_KEY")
  
  ## register rsconnect pin board
  board_register("rsconnect",
                 server = "<rsconnect URL>",
                 account = "<account id>",
                 key = connectAPIKey)
  
  ## generate a ref for the comment
  comment_ref <- stringi::stri_rand_strings(n = 1, length = 12)
  
  comment <- c(
    body,
    list(
      ref = comment_ref,
      date = lubridate::now()
    )
  )
  
  ## check for comments table and create if not present
  if (nrow(pins::pin_find("blog_comment_table_dt", board = "rsconnect")) == 0) {
    comment_tree <- Node$new("comments")
  } else {
    comment_tree <- pins::pin_get(name = "blog_comment_table_dt", board = "rsconnect") 
  }
  
  ## does site_id child node exist?
  if (is.null(FindNode(comment_tree, comment$site_id))) {
    comment_tree$AddChild(comment$site_id)
  }
  
  ## does page_id child node exist?
  site_node <- FindNode(comment_tree, comment$site_id)
  if (is.null(FindNode(site_node, comment$page_id))) {
    site_node$AddChild(comment$page_id)
  }
  
  ## add new comment
  if (!is.na(comment$parent_ref)) {
    parent_node <- FindNode(site_node, comment$parent_ref)
  } else {
    parent_node <- FindNode(site_node, comment$page_id)
  }
  do.call(parent_node$AddChild, c(list(name = comment$ref), comment))
  
  pins::pin(comment_tree, name = "blog_comment_table_dt", board = "rsconnect")
  
  return(comment)
}


#* Retrieve all comments for a page
#* 
#* @param site site id
#* @param page page id
#* 
#* @serializer unboxedJSON
#* 
#* @get /page_comments_dt
function(site = "site_01", page = "page_01") {
  
  ## RSConnect API Key
  connectAPIKey <- Sys.getenv("CONNECT_API_KEY")
  
  ## register rsconnect pin board
  board_register("rsconnect", 
                 server = "https://rsconnect-prod.dit.eu.novartis.net",
                 account = "liebeha1",
                 key = connectAPIKey)
  
  ## get table and filter
  rtn_subtree <- list()
  if (nrow(pins::pin_find("blog_comment_table_dt", board = "rsconnect")) > 0) {
    
    ## get pinned comment tree
    comment_tree <- pins::pin_get(name = "blog_comment_table_dt", board = "rsconnect")
    
    ## is site in comment tree?
    if (!is.null(FindNode(comment_tree, site))) {
      
      ## is page in comment tree and does it have comments?
      found_page_comments <- FindNode(comment_tree[[site]], page)
      if (!is.null(found_page_comments)) {
        rtn_subtree <- as.list(found_page_comments, 
                               mode = "explicit", unname = TRUE)
      }
      
    }
  }
  return(rtn_subtree)
}
```

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_02.png" caption="data.tree with comments illustrating hierarchy.  Data are nested as comments, replies, replies-to-replies, etc.  For example, page_01 contains two comments: R8VkpR08pQTA (with a reply cVGBQzLRV9pa) and lHcoISddQbJp">}}

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_03.png" caption="output from data.tree illustrating the metadata held at each node">}}

## Webpage / Blog Post with Comments

Any page with comments follows the same approach.  The page includes the javascript functions listed above (*comments.js*), some css styling (*style.css*, see below) and the `comment_form` function (sourced from *comment.R*).  
There are a few things to note in the code below.  

-  The two variables, `site_id` and `page_id`, are needed to identify comments for the webpage.  Ideally, we'd define them in the yaml header and use them as parameters in the markdown text.  Unfortunately, when using `render_site`, markdown parameters are not rendered (see [open GitHub issue](https://github.com/rstudio/rmarkdown/issues/903)).  `site_id` and `page_id` are therefore defined within a chunk.
-  The javascript function `update_comments_dt` does not sit in a javascript chunk (you can include javascript in rmarkdown by including a chunk with **js** instead of **r** in the chunk header).  Instead, the code is placed directly within a `<script>` tag.  When processed this way, we can access variables (`site_id` and `page_id`) stored in **r** language chunks earlier in the document.  

```r
    ---
    title: "article 1"
    description: |
      Blog post #1.
    author:
      - name: Harvey Lieberman
    date: 03-03-2022
    output:
      distill::distill_article:
        self_contained: false
    ---


    ```{r setup, include=FALSE}
    knitr::opts_chunk$set(echo = FALSE)
    ```

    ```{r}
    ## define site and page
    page_id <- "page_01"
    site_id <- "site_02"

    ## add function, css and js to page
    source(here::here("comment_dt.R"))
    htmltools::includeCSS(here::here("style.css"))
    htmltools::includeScript(here::here("comment_dt.js"))
    ```

    This is a typical blog post but with a comment section added.  
    Comments include nested replies.


    ```{r}
    ## include comment form
    comment_form_dt(page_id = page_id, site_id = site_id)

    ## js below placed in script tags so that R variable can be included
    ```

    <script>
    update_comments_dt(page_id = "`r page_id`", site_id = "`r site_id`")
    </script>

```

## css

The style.css file takes care of styling comments.  The file is included below.

```css
.comments {
  padding: 20px 10px;
  margin: 0;
}

.form-container input[type=submit] {
  background-color: #04AA6D;
  color: white;
  padding: 12px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.form-container input[type=submit]:hover {
  background-color: #45a049;
}

.comment-header {
  font-size: 1.5em;
  line-height: 1.5em;
  font-weight: 400;
  margin-block-start: 1.5em;
  margin-block-end: 1.5em;
}

.comment-header-margin-narrow {
  margin-block-start: 0.5em;
  margin-block-end: 0.5em;
}

.form-contents {
    padding: 10px;
    margin: 10px;
    display: flex;
    flex-direction: row;
    align-items: center;
}

.form-contents .comment-pic {
    display: flex;
    font-size: 3em;
    align-self: flex-end;
}

.form-details {
    display: flex;
    flex-direction: column;
    flex: 2 1 auto;
}

.form-details input[type=text] {
    border-top: 0px;
    border-bottom: 1px solid #ccc;
    border-left: 0px;
    border-right: 0px;
    outline: 0px;
    padding: 0;
    margin-top: 20px;
    margin-left: 20px;
    font-weight: normal;
}

.form-details input[type=text]:focus {
    border-color: #04AA6D;
    border-width: 2px;
}

.comment-comments input[type=text]{
    width: 90%;
}

.comment-user {
    display: flex;
    flex-direction: row;
}

.comment-short {
    width: 50%;
}

.comment-short input[type=text]{
    width: 80%;
}

.button-container {
    display: flex;
    align-self: flex-end;
}

.button-container input[type=submit] {
  margin: 2px 5px;
  float: right;
}

.comment-holder {
  margin-top: 50px;
}

ul.comment-list {
  list-style: none;
  position: relative;
  padding: 0;
  border: 1px solid #ccc;
}

li.comment-item {
  padding: 20px 10px;
  margin: 20px 0;
  position: relative;
}

.comment-top {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
}

.comment-name {
  font-size: 1.5em;
  font-weight: 400;
  margin: 5px 0;
  color: #5d5d5d;
  align-self: flex-start;
}

.date-holder {
  color: #5d5d5d;
  align-self: flex-end;
  display: inline-flex;
  align-items: baseline;
}

.comment-date {
  font-size: 1em;
  font-weight: 400;
  margin: 5px 0 5px 10px;
}

.comment-text {
  display: block;
  margin: 0 0 10px 0;
}
```

## Output

The follow screen captures illustrate adding comments and replies.

### First comment add to a blog post

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_04.png" >}}

### First comment add to a blog post

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_05.png" >}}

### Adding a reply added to comment #1
Clicking on the *Reply* dropdown opens a reply window

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_06.png" >}}

### Once the reply is added it also includes a dropdown for nesting replies

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_07.png" >}}

### Reply for comment #2 dropdown opened

{{< figure src="/images/post-images/2022-03-05-distill_comments_replies/distill_comments_replies_08.png" >}}

## Conclusion

RStudio Connect can be used with {pins} to hold nested comments for blog pages.  This demonstrates the huge scope that RStudio Connect can play as a CMS.