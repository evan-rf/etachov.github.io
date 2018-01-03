---
layout:     post
title:      Sentiment of the City
date:       2018-01-02
summary:    Analyzing the sentiment of 2017 State of the City speeches
categories: sotc
---

Sentiment analysis is common way to evaluate emotion in text. Companies use it to understand how customers [feel about their products](https://ymedialabs.com/google-sentiment-analysis-api/), medical researchers use it to [evaluate patient satisfaction](http://www.jmir.org/2013/11/e239/) and political sciencist have used it to assess the [tenor of parliamentary debate](https://link.springer.com/chapter/10.1007/978-3-319-06826-8_4) and measure [response](https://www.aclweb.org/anthology/P/P12/P12-3.pdf#page=127) to State of the Union speeches on Twitter. 

Today, I'll apply this technique to my [State of the City corpus](https://github.com/etachov/state_of_the_city) to answer two questions:

1. In 2017, which mayors gave the most positive and negative State of the City speeches?
2. How does sentiment change over the course of a speech? Can we identify clear positive or negative sections?

If you're just interested in the answers, scroll down to the charts below. If you're interested in the methodology, here's the R code:


```R

## load up libraries

library(tidytext)
library(tidyverse)
library(sentimentr)
library(ggridges)

## import and clean the data from github. i swear i'll turn this into a package in 2018...

# read speech metadata
sofc_meta <- read_csv("https://raw.githubusercontent.com/etachov/state_of_the_city/master/sofc_metadata.csv") %>%
  # filter down to speeches that have been collected from 2017
  filter(collected == "Yes" & year == 2017)

# use the id variable to create a list of file paths
file_paths <- paste0("https://raw.githubusercontent.com/etachov/state_of_the_city/master/text/", sofc_meta$id, ".txt")

# and a function to read the text from github and return a data frame with the text and line numbers
speech_txt <- function(file_path) {

  df <- data_frame(text = read_lines(file_path)) %>%
    mutate(line = row_number(),
           id = gsub("https://raw.githubusercontent.com/etachov/state_of_the_city/master/text/|\\.txt", "", file_path))

  return(df)

}

# map this function over the list of file paths and return a single data.frame
text_raw <- map_df(file_paths, speech_txt) 

# tokenize by sentence
speech_sentences <- unnest_tokens(text_raw, output = "sentences", input = "text", token = "sentences")


```

To score sentiment at the sentence level, I used [Tyler Rinker's `sentimentr`](https://github.com/trinker/sentimentr) package. I chose `sentimentr` because it takes into account valence shifters (e.g. negations like "not good") when calculating the scores. 

```R

## score sentiment

full_sentiment <- sentiment(speech_sentences$sentences,
                            # take the full sentence into account
                            n.before = Inf, n.after = Inf) %>% 
  # element_id is the sentence id from unnest_token
  group_by(element_id) %>%
  # in a few instances, sentimentr::sentiment's tokenization approach disagrees with tidytext::unnest_tokens' with 
  # for those few instances, we'll use summary statistics for each sentence
  summarise(word_count = sum(word_count),
            sentiment = mean(sentiment))


# bind the sentence scores to the original data.frame
speech_sent <- bind_cols(speech_sentences, full_sentiment) %>%
  group_by(id) %>%
  # add some summary variables we'll use for the charts
  mutate(element_id = row_number(), 
         mean_sentiment = mean(sentiment, na.rm = T), 
         median_sentiment = median(sentiment, na.rm = T), 
         sd_sentiment = sd(sentiment, na.rm = T)) %>%
  # join with speech metadata
  left_join(sofc_meta)


```

Now here are the answers to those two questions:

### Q1: Which mayors gave the most positive and negative speeches in 2017? ###



![](/images/2018-01-02-sentiment-comparison.svg)


Based on my reading of the text, the ranking checks out. Mayor Andrew Ginther's [speech in Columbus](https://www.columbus.gov/Templates/Detail.aspx?id=2147494899) focused the city's "success story". While Mayor Bill de Blasio [devoted](https://medium.com/@nycgov/this-is-your-city-6230765d11c) a large part of his speech to New York's afforability crisis.

### Q2: How does sentiment change over the course of a speech? ###


![](/images/2018-01-02-sentiment-change-during-speech.svg)

The sentence-to-sentence scores are noisy so I use local regression to find positive and negative sections. The chart highlights a couple of interesting patterns:

1. Speeches tend to start and end relatively positive.
2. Many speeches have distinct negative sections, most clearly Philadelphia and Houston. 

Let's take a closer look at the high and low points of Mayor Jim Kenney's [speech](https://beta.phila.gov/press-releases/mayor/mayor-kenney-delivers-second-chamber-of-commerce-address/) in Philadelphia to ground-test the results:

![](/images/2018-01-02-sentiment-change-during-speech-philadelphia.svg)

Here's sentences 110 - 114:

*"To give start-ups another boost, we also worked with the state to expand the boundaries of the Keystone Innovation Zone to include the growing tech community in Old City. In 2016, 36 companies in this zone benefited from a cumulative $2.4 million in tax credits. Thanks to these and other efforts, the U.S. Chamber of Commerce ranked Philadelphia among the top ten cities for its ability to take advantage of the digital economy, outranking both New York and DC. Overall, I’m pleased to say that our efforts to make Philadelphia more business friendly have paid off."*

And here's sentences 138 - 141:

*"...the opioid crisis and other factors are forcing more people onto the street every day, so we will need to invest more in stemming the tide of the recently-homeless. As part of that strategy, I will ask Council to allocate more of our upcoming budget towards treating drug addiction. Philadelphia is facing an opioid crisis so gripping that we lost 900 people to overdoses last year.  Not to mention the unknown number lost to families and friends, living along the railroad tracks in Kensington or on the floor of a rundown building."*


The code for the analysis and charts is available on [github](https://github.com/etachov/sentiment_of_the_city). And if you'd like to use, edit, or contribute to the State of the City corpus, it's also on [github](https://github.com/etachov/state_of_the_city).

