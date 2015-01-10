---
layout: post
title:  "Export datasets into excel with multiple sheets"
date:   2014-08-07
---


####An R function to write multiple table into excel.
Very often, when I want to save my dataset as a spreadsheet, I export them one by one as separate .csv file. Today, 
I generated multiple really close but different subsets of data, so I want to put them into a single excel file for 
my report. Then I googled and found this method. Besides, the author mentioned a R package [xlsx][2] can do all sorts 
of works related to excel. 

> One solution was to write a wrapper for write.xlsx() function in the xlsx package. (from [r-blooger.com][1], by Rob Kabacoff)

```r
save.xlsx <- function (file, ...)
  {
      require(xlsx, quietly = TRUE)
      objects <- list(...)
      fargs <- as.list(match.call(expand.dots = TRUE))
      objnames <- as.character(fargs)[-c(1, 2)]
      nobjects <- length(objects)
      for (i in 1:nobjects) {
          if (i == 1)
              write.xlsx(objects[[i]], file, sheetName = objnames[i])
          else write.xlsx(objects[[i]], file, sheetName = objnames[i],
              append = TRUE)
      }
      print(paste("Workbook", file, "has", nobjects, "worksheets."))
}

save.xlsx("myworkbook.xlsx", mtcars, Titanic, AirPassengers, state.x77)
```
> should save the R objects mtcars (a data frame),  Titanic (a table),  AirPassengers (a time series) 
and state.x77 (a matrix) to the workbook myworkbook.xlsx. Each object should be in it’s own worksheet 
and the worksheet should take on the name of the object.


[1]: http://www.r-bloggers.com/quickly-export-multiple-r-objects-to-an-excel-workbook
[2]: http://cran.r-project.org/web/packages/xlsx/index.html