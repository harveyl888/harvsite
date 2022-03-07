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
