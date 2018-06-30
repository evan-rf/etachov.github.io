---
layout:     post
title:      Footprint of the City
date:       2018-06-30
summary:    
categories: spatial
---

![](/images/2018-06-30-building-boundaries-south-mall.png)

The Bing Maps team at Microsoft recently [released](https://blogs.bing.com/maps/2018-06/microsoft-releases-125-million-building-footprints-in-the-us-as-open-data/) a dataset of nearly 125 million building footprints across United States.

These footprints were derived using a [two-step process](https://github.com/Microsoft/USBuildingFootprints). First, the team used [deep residual neural network](https://github.com/KaimingHe/deep-residual-networks) to identify likely building pixels. Second, they developed a novel method for grouping the building pixels into sensible polygons.

I plotted building footprints around Mall in Washington DC over satellite imagery for a quick visual inspection: 

<iframe width='100%' height='700px' frameBorder='0' src='http://etachov.io/projects/2018-06-30-building-boundaries-map.html'></iframe>
<center><i>Explore the full screen map <a href = "http://etachov.io/projects/2018-06-30-building-boundaries-map.html" target = "_blank">here</a></i></center><br>

Spend a few minutes exploring the map and you'll see (1) that Bing Maps team's results quite good and (2) there are still anomalies that demonstrate why this is such a hard problem. Notably, the two-step approach struggles with irregular buildings. 

**Hirshhorn Museum**

![](/images/2018-06-30-building-boundaries-hirshhorn.gif)

**Department of Housing and Urban Development**

![](/images/2018-06-30-building-boundaries-hud.gif)

**Watergate Hotel**

The algo did a pretty good job on this one given the complexity.

![](/images/2018-06-30-building-boundaries-watergate.gif)

Edge cases aside, these are incredibly promising results and boon for Open Street Map. I'm looking forward to seeing if/when the Bing Maps team releases datasets for other countries. That data would be extremely helpful for a number development sector problems from poverty estimates to disaster response.

The code to download and map the building footprints is below. As an aside, I recently made the jump and switched my R spatial workflow over from sp to the [sf](https://github.com/r-spatial/sf/). It's been great and I honestly haven't looked back once.

``` r 
library(tidyverse)
library(sf)
library(tigris)
library(mapview)


# download and unzip data from https://github.com/Microsoft/USBuildingFootprints
url_dc <- "https://usbuildingdata.blob.core.windows.net/usbuildings/DistrictofColumbia.zip"
download.file(url_dc, "DistrictofColumbia.zip")
unzip("DistrictofColumbia.zip")

# read in building shapes
dc_buildings <- st_read("/Users/evan/Dropbox/Projects/2018-q2-building-outlines/data/DistrictofColumbia.json") %>%
  # reproject the file to ESPG 3857 (ugh) to match the mapview layer used below
  st_transform(., crs = 3857)

# use the tigris package to get the census tract shapefile
options(tigris_class = "sf") # defaults to sp; manually set to sf
dc_tracts <- tracts(state = "DC")

# filter down to census tracts around the mall 
mall_tracts <- dc_tracts %>%
  filter(NAME %in% c(56, 58, 59, 62.02,  65, 66, 82, 101, 102, 105, 107, 108))

# reproject the mall_tracts
mall_tracts_t <- st_transform(hirsh_tracts, st_crs(dc_buildings))

# clip buildings file to just mall_tracts boundaries
dc_building_select <- st_intersection(mall_tracts_t, dc_buildings)

# make a quick interactive map for inspection
mapview(dc_building_select, 
        col.regions = "coral", 
        color = "white", 
        map.type = "Esri.WorldImagery")

```

