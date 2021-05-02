+++
author = "Harvey"
title = "Extracting data from LCMS PDF"
date = "2019-08-26"
description = "Extracting data from Agilent LCMS PDF"
tags = ["R", "Chemistry"]
+++

Here's an example of using R to extract tabular content and metadata from a series of pdf files and push the data into a single table.  The concept is fairly straightforward - read in PDF and convert to text, parse each line and use regular expressions to extract pertinent information.  

Three types of data are returned:

-  peak data.  Peak number, retention time, type, width, area and name.  In fact all columns will be extracted from the table along with the column identifiers and units.
-  metadata.  In this case injection date, data file name and sample name are pulled from the pdfs.
-  file data.  The pdf file name and page containing the peak table.

The `pdftools` library is used to read in the PDF files.

#### Unwanted or Missing spaces

So why use column identifiers from the table header instead of just splitting table columns using a delimeter such as a tab?  Unfortunately tabular information is lost when the pdf is imported and any tabs are converted into spaces.  A space character column delimeter could be used if all columns are populated and the sample names did not contain spaces.  For these tables, however, the *Type* column is not always populated and the *Name* column can contain spaces.  Therefore there is no way to be sure that we are accurately identifying the end of one column and beginning of another using delimeters.  

An additional issue arises because when the pdf is imported some unexpected formatting issues can arise, particularly with respect to unwanted or missing spaces.  To account for this when parsing the table we can nudge forward or back to identify spaces between values.  This can be seen in the alignment of line 3 of the table below (pdf vs parsed text).

{{< figure src="/images/post-images/2019-08-26-extract_pdf/pdf_table_01.png" title="PDF Version" >}}

{{< figure src="/images/post-images/2019-08-26-extract_pdf/pdf_table_02.png" title="Parsed Version" >}}

#### Code

```r
## table_extract
## extract table contents along with headers
## this allows sample names to be retrieved too
## uses heading underline separators to determine column widths

library(pdftools)
library(stringr)
library(dplyr)

folder <- 'test_01'

## identify files
f_names <- list.files(path = folder, pattern = '.pdf', full.names = TRUE)

## loop over all files
l.df <- lapply(f_names, function(f) {
  
  txt <- pdf_text(f)
  
  ## split text at carriage return
  txt1 <- strsplit(txt, '\r\n')
  
  ## loop over lines and retrieve table contents
  l <- list()
  l_all <- list()
  read_table <- FALSE
  for (i in 1:length(txt1)) { # loop through pages
    for (j in 1:length(txt1[[i]])) { # loop through lines
      line <- txt1[[i]][j]
      
      if (str_detect(line, 'Totals :')) { # end of record
        read_table <- FALSE
        df.data <- as.data.frame(do.call(rbind, l_data), stringsAsFactors = FALSE)
        df.data$date <- l[['date']]
        df.data$data_file <- l[['data_file']]
        df.data$sample_name <- l[['sample_name']]
        df.data$pdf_file <- basename(f)
        df.data$page <- i
        l[['data']] <- df.data
        l_all[[length(l_all) + 1]] <- l
        l <- list()  ## reset list
      } else if  (str_detect(line, 'Injection Date')) { # found inj date
        inj_date <- as.POSIXct(str_extract(line, '\\d+/\\d+/\\d+ \\d+:\\d+:\\d+ [A|P]M'), format = '%m/%d/%Y %I:%M:%S %p')
        l$date <- inj_date
        l_data <- list()
      } else if (str_detect(line, 'Peak RetTime')) { # found table
        table_headers <- line   ## save table headers for later use
        read_table <- TRUE
      } else if (str_detect(line, 'Data File')) {
        l[['data_file']] <- basename(txt1[[2]][1])
      } else if (str_detect(line, 'Sample Name:')) {
        l[['sample_name']] <- str_replace(line, 'Sample Name: ', '')
      }
      
      if (read_table) {
        
        if (str_detect(line, '\\[min\\]')) { # found second header line
          table_headers_2 <- line   ## save table headers for later use
        }
        
        if (!str_detect(line, '\\[min\\]') & !str_detect(line, 'Peak RetTime')) {
          if (str_detect(line, '--|--')) { ## found separator line
            locate_separators <- str_locate_all(line, '\\|')
            locate_separators <- c(1, locate_separators[[1]][,1], nchar(line))
            txt_headings <- sapply(seq_along(locate_separators[-1]), function(x) {
              trimws(substring(table_headers, locate_separators[x], locate_separators[x+1]))
            })
            
            txt_headings_2 <- sapply(seq_along(locate_separators[-1]), function(x) {
              trimws(substring(table_headers_2, locate_separators[x], locate_separators[x+1]))
            })
            
            txt_headings <- sapply(seq_along(txt_headings), function(x) {
              if (nchar(txt_headings_2[x]) > 0) {
                sprintf('%s (%s)', txt_headings[x], txt_headings_2[x])
              } else {
                txt_headings[x]
              }
            })
            
          } else {  ## found data
            
            ## add a space to the end of the line
            ## necessary for identifying end-of-data points
            line <- paste0(line, ' ')
            
            data <- sapply(seq_along(locate_separators[-1]), function(x) {
              
              ## sometimes text pull needs to start a few characters back
              ## (due to import conversion of tabs to spaces)
              start_pos <- locate_separators[x]
              end_pos <- locate_separators[x+1]
              
              if (substring(line, start_pos, start_pos) != ' ') {
                found_space <- FALSE
                while (!found_space & start_pos > 2) {
                  start_pos <- start_pos - 1
                  if (substring(line, start_pos, start_pos) == ' ') found_space <- TRUE
                }
              }
              
              if (substring(line, end_pos, end_pos) != ' ') {
                found_space <- FALSE
                while (!found_space & end_pos > 2) {
                  end_pos <- end_pos - 1
                  if (substring(line, end_pos, end_pos) == ' ') found_space <- TRUE
                }
              }
              
              trimws(substring(line, start_pos, end_pos))
            })
            l_data[[length(l_data) + 1]] <- setNames(data, txt_headings)
          }
        }
      }
    }
  }
  
  ## join tables
  df <- do.call(rbind, lapply(l_all, function(x) x$data))
  
  ## convert table columns to numeric
  for (n in names(df)) {
    if (all(grepl("^(-|\\+)?((\\.?\\d+)|(\\d+\\.\\d+)|(\\d+\\.?)|(\\d*\\.*\\d+[E|e](-|\\+)\\d+))$", df[[n]]))) {
      df[[n]] <- as.numeric(df[[n]])
    }
  }
  df
})

df_all <- do.call(rbind, l.df)

write.csv(df_all, 'table_out.csv', row.names = FALSE)

```

#### Input File Example

{{< figure src="/images/post-images/2019-08-26-extract_pdf/input_file.png" >}}

{{< figure src="/images/post-images/2019-08-26-extract_pdf/output_table.png" >}}
