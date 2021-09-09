+++
author = "Harvey"
title = "{distill} Contact Form"
date = "2021-09-08"
description = "Including a contact form on a {distill} site"
tags = ["R"]
draft = true
+++

## Purpose

Can we use use RStudio Connect to build an internal contact form on a {distill} website?  
{distill} is a great R library for building static websites and blogs but has the same limitations of other static websites, namely no server-side programming.  This means that implementing a contact form requires using a third party service.  This can, however, be accomplished when using RStudio Connect via a plumber API.

## Attempt 1

This first attempt works but since we are using a **POST** request a response is always returned.  This means that the webpage is updated with either a *null* or whatever has been returned by the API function.  The output consists of three files with a {distill} R website:

- index.Rmd - a markdown file to hold the contact form
- contact_form.html - an html contact form that can be inserted into a markdown page
- style.css - some css styling (pulled from [w3schools](https://www.w3schools.com/howto/howto_css_contact_form.asp))

and a plumber API:

- plumber.R - a plumber API

Both the {distill} static site and plumber API are published to the same RStudio Connect instance.

### plumber.R

The plumber API takes parameters from a contact form and constructs a linux mail command line, sending a message to a mailbox.  The API contains a single POST request to /email.

~~~r
library(plumber)

#* @apiTitle Email

#* Send out an email message
#* @param email return email address
#* @param name sender name
#* @param subject email subject
#* @param message email message
#* @param to recipient email address
#* @post /email
function(email = NULL, name = NULL, subject = NULL, message = NULL, to = NULL) {
  
  ## need a recipient to send an email
  if (!is.null(to)) {
    
    if (is.null(message)) {
      message <- ""
    } else {
      message <- gsub("\\r", "", message)
    }
    
    ## add sender's name
    if (!is.null(name)) {
      message <- paste0(message, "\n\nFrom: ", name)
    }
    
    ## build up email string
    email_string <- "echo -e "
    email_string <- paste0(email_string, "\"", message, "\" | mail ")
    
    if (!is.null(subject)) {
      email_string <- paste0(email_string, "-s \"", subject, "\" ")
    } else {
      email_string <- paste0(email_string, "-s \"no subject\" ")
    }
    
    if (!is.null(email)) {
      email_string <- paste0(email_string, "-S from=", email, " ")
    }
    
    email_string <- paste0(email_string, to)
    
    system(email_string)
    return(email_string)
  }
}
~~~

### index.Rmd

~~~r
---
title: "Test Contact 1"
---

```{r, echo=FALSE}
htmltools::includeCSS("style.css")

htmltools::includeHTML("contact_form.html")
```
~~~

### contact_form.html

~~~html
<div class="form-container">
  <form action="*url pointing to API*" id="my-form" method="POST">
    
    <label for="name">Name</label>
    <input type="text" id="name" name="name" placeholder="Your name..">
    
    <label for="email">Email</label>
    <input type="text" id="email" name="email" placeholder="Your email address..">
      
    <label for="subject">Subject</label>
    <input type="text" id="subject" name="subject" placeholder="Subject">
                
    <label for="message">Message</label>
    <textarea id="message" name="message" placeholder="Your message.." style="height:200px"></textarea>
  
    <input type="hidden" name="to" value="*mailbox*" />
                    
    <input type="submit" value="Submit">
  
  </form>
</div>
~~~

In the contact_form.html file, *url pointing to API* (line 2) refers to the url of the plumber API, hosted on the same RStudio Connect server as the distill site.  *mailbox* (line 16) refers to the receiving mailbox.  It is included as a hidden element in the form so that it may be passed to the API.

## style.css

~~~css
input[type=text], select, textarea {
  width: 100%; /* Full width */
  padding: 12px; /* Some padding */ 
  border: 1px solid #ccc; /* Gray border */
  border-radius: 4px; /* Rounded borders */
  box-sizing: border-box; /* Make sure that padding and width stays in place */
  margin-top: 6px; /* Add a top margin */
  margin-bottom: 16px; /* Bottom margin */
  resize: vertical /* Allow the user to vertically resize the textarea (not horizontally) */
}

/* Style the submit button with a specific background color etc */
input[type=submit] {
  background-color: #04AA6D;
  color: white;
  padding: 12px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* When moving the mouse over the submit button, add a darker green color */
input[type=submit]:hover {
  background-color: #45a049;
}

/* Add a background color and some padding around the form */
.form-container {
  border-radius: 5px;
  background-color: #f2f2f2;
  padding: 20px;
}
~~~

## Attempt 2

The second attempt builds on the first.  Here the submit button is intercepted by a little javascript function which executes the POST request and captures the output.  Running this way means that the webpage does not update once the request is run.  In addition, we can trigger notification that the form was sent (in this case a simple alert).

Here we have four files with a {distill} R website:

- index.Rmd - a markdown file to hold the contact form
- contact_form.html - an html contact form that can be inserted into a markdown page
- style.css - some css styling (pulled from [w3schools](https://www.w3schools.com/howto/howto_css_contact_form.asp))
- script.js - javascript function to intercept the submit button press

and a plumber API:

- plumber.R - a plumber API

Both the {distill} static site and plumber API are published to the same RStudio Connect instance.

### plumber.R

The plumber API differs from the one in *Attempt 1* by reading a json encoded version of the body.  The API contains a single POST request to /email.

~~~r
library(plumber)
library(jsonlite)

#* @apiTitle Email

#* Send out an email message
#* @param req request body
#* @post /email
function(req) {
  
  ## get the message body
  body <- jsonlite::fromJSON(req$postBody)
  
  email <- body$email
  name <- body$name
  subject <- body$subject
  to <- body$to
  message <- body$message
  
  ## need a recipient to send an email
  if (!is.null(to)) {
    
    if (is.null(message)) {
      message <- ""
    } else {
      message <- gsub("\\r", "", message)
    }
    
    ## add sender's name
    if (!is.null(name)) {
      message <- paste0(message, "\n\nFrom: ", name)
    }
    
    ## build up email string
    email_string <- "echo -e "
    email_string <- paste0(email_string, "\"", message, "\" | mail ")
    
    if (!is.null(subject)) {
      email_string <- paste0(email_string, "-s \"", subject, "\" ")
    } else {
      email_string <- paste0(email_string, "-s \"no subject\" ")
    }
    
    if (!is.null(email)) {
      email_string <- paste0(email_string, "-S from=", email, " ")
    }
    
    email_string <- paste0(email_string, to)
    
    system(email_string)
    return(email_string)
  }
}
~~~

### index.Rmd

The markdown file is very similar to the original, with an additional line to include the javascript file.

~~~r
---
title: "Test Contact 1"
---

```{r, echo=FALSE}
htmltools::includeCSS("style.css")
htmltools::includeScript("script.js")

htmltools::includeHTML("contact_form.html")
```
~~~

### contact_form.html

The contact form does not change significantly from the original, the only difference being the removal of *method="POST"* in the form element.

~~~html
<div class="form-container">
  <form action="*url pointing to API*" id="my-form">
    
    <label for="name">Name</label>
    <input type="text" id="name" name="name" placeholder="Your name..">
    
    <label for="email">Email</label>
    <input type="text" id="email" name="email" placeholder="Your email address..">
      
    <label for="subject">Subject</label>
    <input type="text" id="subject" name="subject" placeholder="Subject">
                
    <label for="message">Message</label>
    <textarea id="message" name="message" placeholder="Your message.." style="height:200px"></textarea>
  
    <input type="hidden" name="to" value="*mailbox*" />
                    
    <input type="submit" value="Submit">
  
  </form>
</div>
~~~

In the contact_form.html file, *url pointing to API* (line 2) refers to the url of the plumber API, hosted on the same RStudio Connect server as the distill site.  *mailbox* (line 16) refers to the receiving mailbox.  It is included as a hidden element in the form so that it may be passed to the API.

## style.css

No change to the style.css file.

## script.js

~~~js
window.addEventListener("load", function() {

  document.getElementById("my-form-2").addEventListener("submit", formsubmit);

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
    });
    
    // notification of sent message
    alert("message sent to " + plainFormData.to);

  }

});
~~~
