+++
author = "Harvey"
title = "Serverless shiny app embedded in a Quarto Website"
date = "2023-10-25"
description = "Serverless shiny app embedded in a Quarto Website"
tags = ["R", "shiny"]
codeMaxLines = 15
draft = true
+++

# Serverless Shiny

Inspired by Max Kuhn's [presentation](https://topepo.github.io/2023-r-pharma/#/title-slide) at R/Pharma today I tried out embedding shiny a app within a Quarto website and hosting it on GitHub pages.  The app itself is a chemistry quiz built to help my son who had a school test coming up.  By running it within a Quarto website it's available for him to use on the bus on the way to school for some quick revision.

{{% notice note "Note" %}}
Thanks so the app he scored 27/30!
{{% /notice %}}

## Setup

To use shinylive in a Quarto document, you need to first install the shinylive Quarto extension:

```bash
quarto add quarto-ext/shinylive
```

along with the shinylive R package

```r
remotes::install_github("posit-dev/r-shinylive")
```


## _quarto.yml file

The _quarto.yml file is a project with some basic parameters to define the website.  The only additional parameter of note is to include `shinylive` as a filter.

```yaml
project:
  type: website
  output-dir: docs
  
format: 
  html
  
website:
  title: "shinylive elements quiz"
  navbar:
    left:
      - index.qmd

filters:
  - shinylive
```

## index.qmd

The shiny app is in a single chunk of the *index.qmd* file with a code chunk type of `shinylive-r` and the `standalone` option set.

~~~
---
title: "Chem symbol test"
---

This is a simple chemical symbol test.  You'll be tested on chemical elements and their names.

```{r}
#| include: false
library(shinylive)
```

```{shinylive-r}
#| viewerHeight: 300
#| standalone: true

chem <- data.frame(
  symbol = c("Al", "Sb", "Ar", "As", "Ba", "Be", "Bi", 
             "B", "Br", "Cd", "Ca", "C", "Cs", "Cl", "Cr",
             "Co", "Cu", "F", "Au", "He", "H", "I", "Fe",
             "Kr", "Pb", "Li", "Mg", "Mn", "Hg", "Ne", "Ni",
             "N", "O", "P", "Pt", "K", "Rn", "Se", "Si", "Ag",
             "Na", "Sr", "S", "Te", "Sn", "Ti", "W",
             "U", "V", "Xe", "Zn", "Zr"),
  name = c("aluminum", "antimony", "argon", "arsenic", "barium", "beryllium", "bismuth", 
           "boron", "bromine","cadmium", "calcium", "carbon", "cesium", "chlorine", "chromium",
           "cobalt", "copper", "fluorine", "gold", "helium", "hydrogen", "iodine", "iron",
           "krypton", "lead", "lithium", "magnesium", "mangenese", "mercury", "neon", "nickel",
           "nitrogen", "oxygen", "phosphorus", "platinum", "potassium", "radon", "selenium", "silicon", "silver",
           "sodium", "strontium", "sulfur", "tellurium", "tin", "titanium", "tungsten",
           "uranium", "vanadium", "xenon", "zinc", "zirconium")
)

ui <- fluidPage(
  fluidRow(
    column(5, offset = 1, checkboxGroupInput("chk_options", "Test me on", choices = names(chem), selected = "symbol", inline = TRUE))
  ),
  fluidRow(
    column(5, offset = 1, uiOutput("ui_question")),
    column(5, 
           fluidRow(uiOutput("ui_score")),
           fluidRow(uiOutput("ui_streak"))
           )
  )
  
)

server <- function(input, output, session) {

  last_num <- 0
  rv <- reactiveValues(
    question_count = 0,
    score = 0,
    question_type = NULL,
    question = NULL,
    answer = NULL,
    streak = 0
  )
  
  question_index <- reactive({
    rv$question_count
    num <- last_num
    while (num == last_num) {
      num <- sample(nrow(chem), size = 1)
    }
    last_num <- num
  })
  
  output$ui_question <- renderUI({
    if (length(input$chk_options) > 0) {
      rv$question_type <- sample(input$chk_options, size = 1)
      answer_type <- names(chem)[!names(chem) == rv$question_type]
      rv$question <- chem[[rv$question_type]][question_index()]
      rv$answer <- chem[[answer_type]][question_index()]
      tagList(
        h3(paste(answer_type, "for:", rv$question)),
        textInput("txt_answer", label = NULL),
        actionButton("but_answer", "Submit")
      )
    }
  })
  
  observeEvent(input$but_answer, {
    if (rv$question_type == "symbol") {
      answer <- tolower(input$txt_answer)
    } else {
      answer <- input$txt_answer
    }
    if (answer == rv$answer) {
      rv$score <- rv$score + 1
      rv$streak <- rv$streak + 1
      showNotification(ui = "CORRECT!", type = "message")
    } else {
      rv$streak <- 0
      showNotification(ui = paste0("WRONG.  Correct answer is ", rv$answer), type = "error")
    }
    rv$question_count <- rv$question_count + 1
  })
  
  output$ui_score <- renderUI({
    h3(paste0("Score: ", rv$score, "/", rv$question_count), style = "color: #388E3C")
  })
  
  output$ui_streak <- renderUI({
    if (rv$streak > 29) {
      col <- "#2E7D32"
    } else if (rv$streak > 19) {
      col <- "#00838F"
    } else if (rv$streak > 9) {
      col <- "#1E88E5"
    } else if (rv$streak > 0) {
      col <- "#8E24AA"
    } else {
      col <- "#E53935"
    }
    h3(paste0("Streak: ", rv$streak), style = paste0("color: ", col))
  })

}

app <- shinyApp(ui = ui, server = server)

```
~~~

The app itself contains the typical *ui* and *server* along with the `shinyapp()` function to execute.  It is basic and unoptimized (but it demonstrates the purpose).  It takes very little effort to change a standalone app to an embedded one.

The quarto website with embedded app is available at https://harveyl888.github.io/shinylive-test/
