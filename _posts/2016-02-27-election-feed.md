---
layout:     post
title:      When's the Next Election in Comoros? 
date:       2016-02-27
summary:    Parsing a semi-hidden feed upcoming elections data
categories: munging
---

The International Foundation for Electoral Systems (IFES) does the
incredible service of maintaining a calendar of upcoming elections on
[electionguide.org](http://www.electionguide.org/).

Their search and filter UI is great if you're interested in a single country, but what if you want to check on 40 countries?

Thankfully there's an straightforward way to do this with R.

IFES maintains a (little advertised) RSS feed, which we can access, parse into a data.frame, and subset by a of list countries. The full code is available on [Github](https://github.com/etachov/election-feedr) and I walk through the steps below:

**1. Find the feed:**

When I first visited electionguide.org, I missed the RSS feed because it's only advertised near the bottom of the site's home page and I'd landed on the [Upcoming Elections](http://www.electionguide.org/elections/upcoming/) page via search.

I ultimaetly located the feed by searching "RSS site:electionguide.org".

**2. Parse the XML:**

RSS feeds are XML data so we'll use the R <code>XML</code> package to parse the feed into an XMLInternalDocument and extract node data using a simple helper function. For more details on XML and R, check out this [great presentation](http://gastonsanchez.com/stat133/slides/33-parsing-xml/33-parsing-xml.pdf) by [Gaston Sanchez](http://gastonsanchez.com/).

    library(dplyr) # general data manipulation
    library(XML) # parse the xml feed
    library(countrycode) # working with country names is tough so i like to convert to codes

     ## parse the raw xml from IFES's Election Guide RSS feed into the R data type XMLInternalDocument
    feed_raw <- xmlParse("http://www.electionguide.org/feed/calendar/upcoming", encoding = "UTF-8")

     ## make a list of the nodes we want to pull out
    node_list <- c("//item/title", "//item/link", "//item/description", "//item/pubDate")

     ## helper function using xpathSApply to extact the value from each node using xmlValue
    nodeGet <- function(x) {
      x <- xpathSApply(feed_raw, x, xmlValue)
      x
    }

     ## apply the function to the list of nodes and turn the result into a data.frame
    elect_raw <- as.data.frame(lapply(node_list, nodeGet)) 

     ## set the names using the original list
    names(elect_raw) <- gsub("//item/", "", node_list)

**3. Clean and subset for countries of interest:**

This leaves us with a messy data.frame that we'll clean up using <code>dplyr</code> and regular expressions. Once you have the tidy data.frame you can subset by a vector of countries and write it out as a .csv file or upload it to Google Sheets with the very helpful <code>googlesheets</code> package from [Jenny Bryan](https://github.com/jennybc/googlesheets)

     ## clean up the data with some regex and dplyr
    elect <- elect_raw %>%
      mutate(country = gsub(":.*", "", title),
             iso3c = countrycode(country, "country.name", "iso3c"),
             elect.type = gsub(".*:", "", title),
              # remove the html from the description
             description = gsub("<.*?>|&nbsp;", "", description),
              # because i always forget the symbols for the format function http://statmethods.net/input/dates.html
             date = as.Date(pubDate, format = "%a, %d %b %Y")) %>%
      select(country, iso3c, elect.type, date, description, link)


     ## subset the list by a vector of countries if you have a specific focus: 
    countries_interest <- countrycode(c("Comoros", "Zambia", "United Kingdom"), "country.name", "iso3c")

    elect %>%
      filter(iso3c %in% countries_interest)

I use this script to keep tabs on elections in countries where <a href = "http://www.mdif.org" target = "_blank">MDIF</a> has investments. When an election takes place, we evaluate how our client's coverage compares with other media sources visualize the results results on our <a href = "http://www.mdif.org/client-election-coverage/" target = "_blank">Election StoryMap</a>.

Finally, for those that read all this way, Comoros' next election (as of this post) is a [presidential contest](https://en.wikipedia.org/wiki/Comorian_presidential_election,_2016) in April.