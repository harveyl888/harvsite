+++
author = "Harvey"
title = "Building a Quarto Extension"
date = "2023-11-20"
description = "First attempt at building a Quarto extension"
tags = ["Quarto"]
codeMaxLines = 15
draft = false
+++

# Building my First Quarto Shortcode Extension

[Quarto](https://quarto.org/) is a scientific publishing system used to build documents, presentations, books and websites.  Quarto extensions are a powerful way to extend the behavior of Quarto.  These extensions are built using Lua, a language that I do not have much familiarity with but, building a simple extension is actually quite straighforward.

In this example I'll create a shortcode extension that allows you to add comments to a Quarto document.  The comments can be turned on or off by using an environemnt variable defined in 

## Initialization

To initialize a Quarto shortcode extension, use the following command and enter a name for the extension (in this case *comment*)

```
quarto create extension shortcode
```

The following folder structure will be created:

```
comment
├── _extensions  
│   └── comment  
│       ├── _extension.yml  
│       └── comment.lua  
├── .gitignore  
├── .luarc.json  
├── example.qmd
└── README.md  
```

I use VS Code for coding.  When the `quarto create` command is executed, VS Code opens a new window at the `comment` folder, ready for development.

## Building the Shortcode Extension

This Quarto extension allows an author to add comments to a Quarto document.  It also adds a parameter to control if comments should included on not when a file is rendered.  I've found this extension useful when adding instructional comments to a template.

The extension contains three shortcodes.  One to start a comment block, one to end a comment block and one to add an inline comment.  
The code concept is pretty simple.  For comment blocks, surround the contents in a div with either the `cmt` class (which can be used to control styling) when the comment variable is set to true, or `display: none` when the comment variable is set to false.  For inline comments surround the contents in a span with the `cmt` class when the comment variable is set to true.

The `comment.lua` file is shown below.

```lua
-- start a comment block
function comment_start(args, kwargs, meta)
  cmt = meta['show_comments']
  if cmt == nil or cmt == false then
    return pandoc.RawBlock('html', "<div style = 'display: none;'>")
  else
    return pandoc.RawBlock('html', "<div class = 'cmt'>")
  end
end

-- end a comment block
function comment_stop()
  return pandoc.RawBlock('html', "</div>")
end

-- inline comment
function comment(args, kwargs, meta)
  txt = pandoc.utils.stringify(args[1])
  cmt = meta['show_comments']
  if cmt == true then
    return pandoc.RawInline('html', "<span class = 'cmt'>" .. txt .. "</span>")
  end 
end
```

In addition to the `comment.lua` file, an `example.qmd` should be written (plus, in this case, a `style.css` css file).  If the `example.qmd` file is previewed then it will automatically update as you edit comment.lua - a great way to ensure that your extension is working as expected.  The `example.qmd` and `style.css` files for the comment extension are shown below:

```md
---
title: "Comment Example"
format:
  html:
    css: style.css
show_comments: true
---

## Comments Test - Commented text within a paragraph

This is some uncommented text.
{{</* comment_start */>}}
Here is a <b>comment</b> containing some instructional information.
{{</* comment_stop */>}}
Finally, some additional uncommented text.

## Comments Test - Inline commenting

This comment is an inline comment {{</* comment "It can include instruction within the text" */>}} followed by addititional text.
```

```css
.cmt {
  color: #AAAA00
}
```

To run in interactive / preview mode simple execute the command:

```bash
quarto preview example.qmd
```

## Exploring the Files

The `example.qmd` file contains some example Quarto to test the extension along with a header.  The header applies styles in the `style.css` file (the *.cmt* class) and also defines a boolean parameter called `show_comments`.  The `comment.lua` file contains three functions, each evaluating to a shortcode in Quarto.  They are each explored below.

### comment_start
This shortcode starts a comment block.  It takes three arguments, args, kwargs and meta (arguments, named arguments and document/project-level metadata).  In this function we only use the document/project-level metadata and run the command `cmt = meta['show_comments']` to create a variable, `cmt` holding the metadata `show_comments`, a boolean.  Generally, arguments passed to lua will be a list of pandoc inlines and require `pandoc.utils.stringify()` to convert to strings.  In our case, we are only passing a single value in `show_comments` and it will be a boolean.  
If the `cmt` variable is false or missing then a *div* block is started with `display: none` to hide it.  If the `cmt` variable is true then a *div* block is started with the *.cmt* style. 

### comment_stop
This simply closes the *div* initiated by `comment_start()`.

### comment_inline
This shortcode adds a comment inline.  It works slightly differently to `comment_start()` by reading the first argument from the shortcode using `txt = pandoc.utils.stringify(args[1])`.  It then defines the `cmt` variable as in `comment_start()` and, if `cmt` is true, outputs the commented text, returning nothing if `cmt` is not true.

## Extending

In this example, I've included `show_comments` as a document-level parameter.  It could be substituted at the project-level in `_quarto.yml` to manage a set of documents / website.  It can also be included in `_variables.yml` as a project-level variable in which case the line `cmt = meta['show_comments']` would be replaced by `cmt = meta['_quarto-vars']['show_comments']` in order to access variables defined in the `_variables.yml` file.

## Final Steps

Finally, a licence file was added (MIT licence), README was edited and the extension was pushed to GitHub at https://github.com/harveyl888/comment.  To install to a quarto project simply run: 

```bash
quarto add harveyl888/comment
```

# Conclusion

Quarto includes methods to build all manner of extensions for documents and projects.  It is pretty simple to pick up lua, the scripting language for Quarto extensions, which means writing shortcodes should be fairly straightforward.  The use of an example file along with Quarto's hot reload provides a powerful way to build up shortcodes in an interactive manner[^1].

[^1]: Note: the lua `print` command can be used when developing in an interactive manner - it will print to the terminal whenever the example file is re-rendered.


