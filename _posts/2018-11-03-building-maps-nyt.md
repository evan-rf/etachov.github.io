---
layout:     post
title:      All the Buildings in Glendale, Arizona
date:       2018-10-28
summary:    
categories: spatial
---

![](/images/2018-10-28-glendale_az.png)

Inspired by the recent *New York Times* [map of every building in America](https://www.nytimes.com/interactive/2018/10/12/us/map-of-every-building-in-the-united-states.html) I wrote a quick chunk of R code to produce similar maps for selected zip codes using the same [data from Microsoft](https://etachov.io/spatial/2018/06/30/building-boundaries/). Thanks to some recent advances in the R geospatial ecosystem, i.e. the maturation of the `sf` package and its integration with the `tidyverse`, this was remarkably simple. 

It's been fun to see others apapt the approach to map building in [Evanston, IL](https://twitter.com/hughbartling/status/1055670667209834504), [Boston, MA](https://twitter.com/andrwmllr/status/1056598521384747009), and a number of [cities in France](https://twitter.com/matamix/status/1052450295761051648).


``` r 
library(tidyverse)
library(sf)
library(tigris)

# start by picking a state from https://github.com/Microsoft/USBuildingFootprints
# WARNING: these files can be pretty big. using arizona for its copious subdivisions and reasonable 83MB.
url_footprint <- "https://usbuildingdata.blob.core.windows.net/usbuildings-v1-1/Arizona.zip"
download.file(url_footprint, "Arizona.zip")
unzip("Arizona.zip")

# read in building shapes 
buildings <- st_read("Arizona.geojson")

# use the tigris package to get zctas boundaries (basically zip codes)
options(tigris_class = "sf", tigris_use_cache = TRUE) # defaults to sp; manually set to sf
zctas_national <- zctas(cb = TRUE)

# get the geometry for the zip codes we want to map
selected_zip <- zctas_national %>%
  # input the zipcode(s) you want here. using 85308 for glendale, az
  filter(GEOID10 %in% c(85308)) %>%
  st_transform(st_crs(buildings))

# clip buildings file to just the selected zipcode
building_select <- st_intersection(selected_zip, buildings)

# plot using using geom_sf
glendale <- ggplot() +
  geom_sf(data = building_select, fill = "black", color = NA) +
  theme_void() +
  # manually dropping the grid to solve a bug in geom_sf 
  theme(panel.grid.major = element_line(colour = 'transparent'))

ggsave("glendale_az.png", glendale, dpi = 400, width = 8, height = 5)


```

This work also inspired me to re-read [Charlotte Rost's](https://twitter.com/lisacrost) great speech [on Map Poetry](https://lisacharlotterost.github.io/2016/10/21/mappoetry/):

*When we look at a map of the world, we can’t see it as it is. We all lay an internal map of the world on top of the geographical one and only see what’s important to us. This internal map consists of memories & experience, associations, values, beliefs. It is incredible individual. There is no internal map that matches a second one. We all look at the same world differently. And we all look at the same maps differently, and sees different things; according to our internal map of the world. [...]*

*[...] Of course, each map simplifies. That’s what maps are made for. The map can’t be the territory. We can’t deal with the chaos out there. Both the geographical and the internal map must simplify to be useful. [...]*

*[...] Map Poetry doesn’t give us the overview, how maps usually do it. Map Poetry focuses on a subset of the world and looks at it from a subjective standpoint rather than a seemingly objective god-like perspective. Map Poetry makes us aware that the territory out there is, indeed, complicated, and complex and entangled. And that we all perceive it differently.*