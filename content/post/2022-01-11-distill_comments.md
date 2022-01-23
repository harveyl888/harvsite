+++
author = "Harvey"
title = "{distill} Comments"
date = "2022-01-11"
description = "Including a comments section in {distill} pages"
tags = ["R", "javascript"]
draft = true
+++

Following on from a post on [including a contact form on a {distill} site hosted on RStudio Connect](/post/2021-09-08-distill_contact_form/), here's a post on how to include comments in blog posts.

Commenting consists of two parts - a way to retrieve comments that belong to a page and a form to enter new comments.

## New Comment Form

The function `comment_form` takes `site_id` and `page_id` arguments and returns an HTML form.  The form captures a comment and optional user name and passes each of these, along with `site_id` and `page_id` to a plumber API.  The plumber API updates a pinned data frame with the new comment.  In fact, a javascript function intercepts the submit button triggering an update of the page comments after adding the new one.  This allows a new comment to be added without having to refresh the page manually.  
In addition, the `comment_form` function adds a div with the id `rtncomments` which is a placeholder to display comments.  
The `comment_form` R function along with the javascript eventListener are shown below.  In the code, *<rsconnect URL>/addcomment* refers to the plumber API endpoint for adding a new comment.  

```r
library(htmltools)

comment_form <- function(page_id = 0, site_id = 0) {
  
  comment_html <- paste0('
  <div class="comments">
    <div class="form-container">
      <h3 class="comment-header">Post a Comment</h3>
      <form action="<rsconnect URL>/addcomment" id="my-form">
      
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
  
  document.getElementById("my-form").addEventListener("submit", formsubmit);
  
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
    out = await fetch(form.action, {
      method: 'POST',
      body: jsonFormData,
      headers: {
        "Content-Type": "application/json",
        "Accept": "application/json"
      }
    })
    .then(response => response.json());
    
    // update comments
    update_comments(plainFormData.page_id, plainFormData.site_id);

  };

});
```

### Existing Comments

To retrieve existing comments we use a javascript function to build an HTML output.  The function takes `site_id` and `page_id` arguments and calls a plumber API which returns comments belonging to the page in json form.  
Here, *<rsconnect URL>/page_comments* refers to the plumber API endpoint for retrieving comments.  The search parameters `site_id` and `page_id` are appended to the url so that we can limit the returning data to a specific page on a specific site.  Since we are using fetch, the webpage and API must live on the same RStudio Connect instance.  
Once the json-formatted response is returned, the function loops through the comments building an HTML response and updates `#rtncomments`.  If a comment does not have an author it defaults to *anonymous user*.  

```js
function update_comments(page_id, site_id) {

  const url = "<rsconnect URL>/page_comments?"

  fetch(url + new URLSearchParams({
    site: site_id, 
    page: page_id,
  }))
  .then(response => response.json())  
  .then(data => {
    
    // outer_div - placeholder for comments
    div_outer = $('<div/>').attr('id', 'div_outer');
    
    // add comments count
    div_outer.append('<h3>' + data.length + ' Comments</h3>');

    // loop through returned comments, adding each one to an unordered list
    ul_list_comments = $('<ul/>', {id: 'list_comments', class: 'comment-list'});

    $.each(data, function(i, obj) {
      
      user_name = obj.user_name == "null" ? "anonymous user" : obj.user_name

      ul_list_comments.append(
        $('<li/>', {class: 'comment-item'}).append([
          $('<div/>', {class: 'comment-top'}).append([
            $('<h3/>', {class: 'comment-name', text: user_name}),
            $('<span/>', {class: 'date-holder'}).append([
              $('<i/>', {class: 'far fa-clock'}),
              $('<h3/>', {class: 'comment-date', text: obj.date})
            ])
          ]),
          $('<p/>', {class: 'comment-text', text: obj.comment})
        ])
      );
    });

    div_outer.append(ul_list_comments);

    $("#rtncomments").html(div_outer);

  })
  .catch((err) => console.log("Can’t access " + url + " response. Blocked by browser?" + err));
  
};
```

### plumber API

The {distill} blog pages are connected to the comments via a plumber API.  The API contains two endpoints, a POST endpoint, `addcomment` which is used to add a new comment and a GET endpoint, `page_comments` which is used to retrieve comments for a specific page.  The comments themselves are stored in a data frame which is accessible via the {pins} package.  This allows mutiple sites to use the same data frame.  
Once again, *<rsconnect URL>* refers to the RStudio Connect URL and, *<account id>* is the account associated with the pin.  
In the code below, `board_register("rsconnect", server = "<rsconnect URL>, account = "<account id>", key = connectAPIKey)` registers a pin board which holds a pin called `blog_comments`, a data frame with columns for site_id, page_id, user_id, date and comment.  The date is a timestamp set when the comment is added.

```r
library(plumber)
library(jsonlite)
library(pins)
library(tibble)
library(lubridate)
library(dplyr)

#* @apiTitle Comments

#* Add a comment to the table
#* @param req request body
#* @post /addcomment
function(req) {
  
  ## get the message body
 body <- jsonlite::fromJSON(req$postBody)

  ## RSConnect API Key
  connectAPIKey <- Sys.getenv("CONNECT_API_KEY")

  ## register rsconnect pin board
  board_register("rsconnect",
                 server = "<rsconnect URL>,
                 account = "<account id>",
                 key = connectAPIKey)

  ## check for comments table and create if not present
  if (nrow(pins::pin_find("blog_comments", board = "rsconnect")) == 0) {
    comments <- tibble(
      site_id = body$site_id,
      page_id = body$page_id,
      user_id = body$user_id,
      date = lubridate::now(),
      comment = body$comment
    )
  } else {
    comments <- pins::pin_get(name = "blog_comments", board = "rsconnect") %>%
      add_row(
        site_id = body$site_id,
        page_id = body$page_id,
        user_id = body$user_id,
        date = lubridate::now(),
        comment = body$comment
      )
  }
  pins::pin(comments, name = "blog_comments", board = "rsconnect")
  
}

#* Retrieve all comments for a page
#* @param site site id
#* @param page page id
#* @serializer unboxedJSON
#* @get /page_comments
function(site = 0, page = 0) {
  
  ## RSConnect API Key
  connectAPIKey <- Sys.getenv("CONNECT_API_KEY")
  
  ## register rsconnect pin board
  board_register("rsconnect", 
                 server = "<rsconnect URL>,
                 account = "<account id>",
                 key = connectAPIKey)
  
  ## get table and filter
  pins::pin_get(name = "blog_comments", board = "rsconnect") %>%
    dplyr::filter(site_id == site & page_id == page) %>%
    dplyr::arrange(desc(date))
  
}
```