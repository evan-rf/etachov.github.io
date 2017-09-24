---
layout:     post
title:      Mystery Science Computer Vision
date:       2017-09-17
summary:    Cozy up and watch a movie with CV
categories: AI on AI
---

![](/images/rat_film_thoughts_optimized.gif)
*Film runtime is on the x-axis, the algorithm guess confidence is on the y-axis*

In honor of [Rat Film's](https://memory.is/rat-film/) theatrical release, I showed 100 stills from the movie to [Google's Cloud Vision API](https://cloud.google.com/vision/). Cloud Vision tried to identify objects in the stills. I took its three best guesses for each image and created the gif above with [gganimate](https://github.com/dgrtwo/gganimate).


```R

library(tidyverse)
library(RoogleVision)
library(gganimate)
library(ggrepel)
library(jsonlite)

## import credentials files downloaded from my Google Cloud consule
credentials <- fromJSON("credentials.json")

## authenticate
options("googleAuthR.client_id" = credentials$installed$client_id)
options("googleAuthR.client_secret" = credentials$installed$client_secret)
options("googleAuthR.scopes.selected" = c("https://www.googleapis.com/auth/cloud-platform"))
googleAuthR::gar_auth()


## function to label image and return a data frame
labeleR <- function(directory, file_name) {
  
  dat <- getGoogleVisionResponse(imagePath = paste0(directory, file_name), 
                                 feature = "LABEL_DETECTION", 
                                 numResults = 3) %>%
  mutate(image = file_name)
  return(dat)
}

## a folder of images with ordered files names, e.g. image_t1, image_t2
image_directory <- "/Images/"

# generate a list of files the directory
list_of_images <- list.files(image_directory)

# map the labelR function over the file names in the given directory
rat_film_desc <- map_df(list_of_images, labeleR, directory = image_directory) %>%
  # add a simple index for plotting
  mutate(index = as.numeric(ordered(image)))

## build the plot
p <- ggplot(rat_film_desc, aes(x = index, y = score, label = description)) +
  # base layer of all tags
  geom_text(color = "white", alpha = .3, size = 4, family = "mono") +
  # highlights tags to appear as the gif cycles through the movie
  # using the frame = index to tell gganimate what to animate
  geom_text_repel(aes(frame = index), color = "white", size = 10, family = "mono", segment.alpha = 0)+
  labs(title = "\nRat Film") +
  xlim(-5, 95) +
  ylim(.5, 1)+
  theme_void() +
  theme(plot.background = element_rect(fill = "black"), 
        plot.title = element_text(color = "white", size = 50, family = "mono", face = "bold", hjust = 0.5)) 

## animate the plot
gganimate(p, "rat_film_thoughts.gif", 
          interval = .2, 
          ani.width = 800, 
          ani.height = 600,
          title_frame = F)



```