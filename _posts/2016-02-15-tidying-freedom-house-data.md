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

1. Downloading the Raw Data
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

2. Reshaping the Wide Data Frame
--------------------------------

This raw file has one column for country and then columns to the right
for each score-year combo:

<table>
<thead>
<tr class="header">
<th align="left">country</th>
<th align="right">A</th>
<th align="right">B</th>
<th align="right">C</th>
<th align="right">SCORE</th>
<th align="left">STATUS</th>
<th align="right">A.1</th>
<th align="right">B.1</th>
<th align="right">C.1</th>
<th align="right">SCORE.1</th>
<th align="left">STATUS.1</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Afghanistan</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="left">NA</td>
<td align="right">24</td>
<td align="right">30</td>
<td align="right">20</td>
<td align="right">74</td>
<td align="left">NF</td>
</tr>
<tr class="even">
<td align="left">Albania</td>
<td align="right">24</td>
<td align="right">12</td>
<td align="right">12</td>
<td align="right">48</td>
<td align="left">PF</td>
<td align="right">20</td>
<td align="right">18</td>
<td align="right">12</td>
<td align="right">50</td>
<td align="left">PF</td>
</tr>
<tr class="odd">
<td align="left">Algeria</td>
<td align="right">21</td>
<td align="right">26</td>
<td align="right">15</td>
<td align="right">62</td>
<td align="left">NF</td>
<td align="right">21</td>
<td align="right">24</td>
<td align="right">17</td>
<td align="right">62</td>
<td align="left">NF</td>
</tr>
<tr class="even">
<td align="left">Andorra</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="left">NA</td>
<td align="right">1</td>
<td align="right">1</td>
<td align="right">6</td>
<td align="right">8</td>
<td align="left">F</td>
</tr>
<tr class="odd">
<td align="left">Angola</td>
<td align="right">21</td>
<td align="right">33</td>
<td align="right">25</td>
<td align="right">79</td>
<td align="left">NF</td>
<td align="right">20</td>
<td align="right">30</td>
<td align="right">22</td>
<td align="right">72</td>
<td align="left">NF</td>
</tr>
</tbody>
</table>

To make these data easier to use, we need to convert it into a long
format where each combination of country, year and scores has its own
row. To do this we'll use **reshape2::melt** to first melt the wide file
down to a long file and **tidyr::gather** to gather the scores for each
country-year combo.

    library(tidyr) 
    library(reshape2) # for the melt function

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

And we're done:

<table>
<thead>
<tr class="header">
<th align="left">country</th>
<th align="right">year.collected</th>
<th align="right">year.reported</th>
<th align="right">a</th>
<th align="right">b</th>
<th align="right">c</th>
<th align="right">score</th>
<th align="left">status</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Afghanistan</td>
<td align="right">2001</td>
<td align="right">2002</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="right">NA</td>
<td align="left">NA</td>
</tr>
<tr class="even">
<td align="left">Afghanistan</td>
<td align="right">2002</td>
<td align="right">2003</td>
<td align="right">24</td>
<td align="right">30</td>
<td align="right">20</td>
<td align="right">74</td>
<td align="left">NF</td>
</tr>
<tr class="odd">
<td align="left">Afghanistan</td>
<td align="right">2003</td>
<td align="right">2004</td>
<td align="right">24</td>
<td align="right">28</td>
<td align="right">20</td>
<td align="right">72</td>
<td align="left">NF</td>
</tr>
<tr class="even">
<td align="left">Afghanistan</td>
<td align="right">2004</td>
<td align="right">2005</td>
<td align="right">21</td>
<td align="right">27</td>
<td align="right">20</td>
<td align="right">68</td>
<td align="left">NF</td>
</tr>
<tr class="odd">
<td align="left">Afghanistan</td>
<td align="right">2005</td>
<td align="right">2006</td>
<td align="right">21</td>
<td align="right">28</td>
<td align="right">20</td>
<td align="right">69</td>
<td align="left">NF</td>
</tr>
<tr class="even">
<td align="left">Afghanistan</td>
<td align="right">2006</td>
<td align="right">2007</td>
<td align="right">21</td>
<td align="right">28</td>
<td align="right">20</td>
<td align="right">69</td>
<td align="left">NF</td>
</tr>
</tbody>
</table>



You can download the full data
[here](https://github.com/etachov/tidying-freedom-of-the-press/blob/master/fotp_2001_2014.csv).

Bonus Gif!
----------

![](/images/fotp_2001_2014.gif)

Notice how the bimodal distribution has converged in recent years.
