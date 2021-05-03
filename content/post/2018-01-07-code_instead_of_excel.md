+++
author = "Harvey"
title = "Code instead of Spreadsheet"
date = "2018-01-07"
description = "Why can't we break the Spreadsheet mold in Science?"
tags = ["opinion"]
+++

There are many reasons why data analysis should be performed using a language such as R or python over Excel but I'm constantly fighting with colleagues who refuse to adapt.

Here are five reasons to stop using Excel:

#### Reproducibility
It's very challenging to use what's been done before with a spreadsheet.  You can make additional copies of a workbook but sooner or later the file naming convention is forgotten and you have no idea what you were working on a year ago.

#### Version Control
I've started applying git to all kids of work, not just programming.  From a coding perspective if I can't keep a detailed log of what I've done and branch to try new things I'm lost.  Excel 2016 does seem to have some version control through the History pane but since I use Office 2010 at work it's a moot point.  Version control is essential when you're working with anything that might break.

#### Testing
Without built-in tests how can you be sure youre analyses are correct?  All too often we find systematic mistakes in spreadsheets.  This generally arises from a cut-and-paste operation to an incorrect cell, mixing relative and absolute cell references or not updating a formula from a older spreadsheet version.  Breaking a problem into chunks, programming the chunks as functions and running unit tests on those functions will help eradicate any bugs.
A few favorite examples
-  Panko’s analysis of 13 field audits suggested that 94% of spreadsheets contained errors and 5.2% of cells were questionable. (Stephen G. Powell, Kenneth R. Baker and Barry Lawson, 12/01/2007. “A Critical Review of the Literature on Spreadsheet Errors”. Tuck School of Business at Darthmouth College)
-  The report on J.P. Morgan Chase $6 billion derivatives trading loss points to an unnamed overworked staffer in London who copied and pasted the wrong figures into an Excel spreadsheet, throwing the firm’s risk models out of whack.
-  One in five papers published genetics papers from 2005-2015 (3600 papers, 18 journals) contain errors due to Excel autoformatting [Genome Biology, 2016, 17, 177](https://genomebiology.biomedcentral.com/articles/10.1186/s13059-016-1044-7)

#### Maintainability
I work in Analytical Chemistry and often deal with assay data - essentially a block of data which could be summarized as *s* samples each run *r* times against *f* features.  Changing *r* from two repeat samples to three in Excel is challenging and generally requires adding additional columns and reapplying formulae.  This is much more amenable in a programming paradigm where formulae can be much more flexible and account for future changes.
In addition, it's far easier to write and maintain functions under R or python that Excel.  The last significant change in VBA was around 2000.

#### Numerical Accuracy
Numerical accuracy can be an issue for any computing system.  The way floating point values are stored has always meant that we have to be careful with rounding and comparing to zero.  One particular anomoly has been identified with Excel, however.
An old thread at [https://office-watch.com/2008/excel-sum-anomaly/](https://office-watch.com/2008/excel-sum-anomaly/) highlights a typical floating point issue.  Adding a list of numbers does not exactly equal zero.  This is expected since numbers are stored using a limited number of binary digits.  What's interesting is if you take the same column of numbers and order them numerically (high to low or low to high) they do equate to exactly zero.  It appears that the result is dependent on the order of the numbers and refutes the commutativity of addition!


| |      |      |      |
|-|-----------|-----------|-----------|
| |-127551.73 |-127551.73 |103130.41  |
| |103130.41  |1568.90    |9028.59    |
| |1807.75    |1794.71    |7390.11    |
| |7390.11    |1807.75    |2831.26    |
| |9028.59    |2831.26    |1807.75    |
| |2831.26    |7390.11    |1794.71    |
| |1568.90    |9028.59    |1568.90    |
| |1794.71    |103130.41  |-127551.73 |
|SUM|0.0000000000086402|0.0000000000000000|0.0000000000000000|

