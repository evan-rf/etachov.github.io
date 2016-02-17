---
layout:     post
title:      Tidying Freedom of the Press Data (with gif!)
date:       2016-02-15
summary:    Making Freedom House's data easier to use
categories: cleaning, media
---

Freedom House does a great of job collecting data on the state of
freedom around the world. In my day job, I use their
<a href="https://freedomhouse.org/report/freedom-press/freedom-press-2015" target="_blank">Freedom
of the Press</a> (FOTP) data to understand the environment for
independent media in the countries where we invest.

The only issue is that the FOTP data is only available in as wide Excel
file. In this post I'll describe how to download and manipulate the raw
data into a tidy long file file, which can be easily analyzed or merged
with other data. If you don't care about the process, you can download
the cleaned data
[here](https://github.com/etachov/tidying-freedom-of-the-press/blob/master/fotp_2001_2014.csv).

Downloading the Raw Data
---------------------------

To get the data into R, we'll use the **downloader** library to grab the
file from Freedom House's website and the **readxl** library to import
the .xlsx file.

    library(downloader) # makes downloading from https easy

    #download the raw file from Freedom House's website into the current working directory
    download("https://freedomhouse.org/sites/default/files/FOTP2015%20Detailed%20Data%20and%20Subscores%201980-2015.xlsx", dest="fh_raw.xlsx", mode = "wb") 

    library(readxl) # dealing with excel files
    library(dplyr) # the data manipulation machete

    # read in the contents of the "Global" sheet
    fh_raw <- read_excel("fh_raw.xlsx", sheet = "Global", na = "N/A", skip = 4) %>%
      # select the chunk we need 
      .[c(1:210), c(1,103:172)] %>%
      data.frame() 

    # add a variable name  for the first column
    names(fh_raw)[1] <- "country"

Reshaping the Wide Data Frame
--------------------------------

This raw file has one column for country and then columns to the right for each score-year combo:


|country     |  A|  B|  C| SCORE|STATUS | A.1| B.1| C.1| SCORE.1|STATUS.1 |
|:-----------|--:|--:|--:|-----:|:------|---:|---:|---:|-------:|:--------|
|Afghanistan | NA| NA| NA|    NA|NA     |  24|  30|  20|      74|NF       |
|Albania     | 24| 12| 12|    48|PF     |  20|  18|  12|      50|PF       |
|Algeria     | 21| 26| 15|    62|NF     |  21|  24|  17|      62|NF       |
|Andorra     | NA| NA| NA|    NA|NA     |   1|   1|   6|       8|F        |
|Angola      | 21| 33| 25|    79|NF     |  20|  30|  22|      72|NF       |


To make these data easier to use, we need to convert it into a long
format where each combination of country, year and scores has its own
row. To do this we'll use **reshape2::melt** to first melt the wide file
down to a long file and **tidyr::gather** to gather the scores for each
country-year combo.


    library(tidyr) 
    library(reshape2) 

    fh_clean <- melt(fh_raw, id.vars = c("country")) %>% 
      # add variables for the year collected and the year reported
      mutate(year.collected = rep(c(2001:2014), each = 5*210),
             year.reported = year.collected + 1) %>%
      rename(metric = variable, result = value) %>% 
      # delete the extra info from the variable names using gsub 
      mutate(metric = tolower(gsub("\\..*", "", metric))) %>%
      # and finally spread it back out
      spread(metric, result) %>%
      # convert the 
      mutate_each(funs(as.numeric), a, b, c, score)

    write.csv(fh_clean, "fotp_2001_2014.csv", row.names = F)


And we're done. You can download the full data
[here](https://github.com/etachov/tidying-freedom-of-the-press/blob/master/fotp_2001_2014.csv).


|country     | year.collected| year.reported|  a|  b|  c| score|status |
|:-----------|--------------:|-------------:|--:|--:|--:|-----:|:------|
|Afghanistan |           2001|          2002| NA| NA| NA|    NA|NA     |
|Afghanistan |           2002|          2003| 24| 30| 20|    74|NF     |
|Afghanistan |           2003|          2004| 24| 28| 20|    72|NF     |
|Afghanistan |           2004|          2005| 21| 27| 20|    68|NF     |
|Afghanistan |           2005|          2006| 21| 28| 20|    69|NF     |
|Afghanistan |           2006|          2007| 21| 28| 20|    69|NF     |





Bonus Gif!
----------

![](/images/fotp_2001_2014.gif)
