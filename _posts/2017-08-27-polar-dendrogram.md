---
layout:     post
title:      Polar Dendrogram
date:       2017-08-27
summary:    
categories: photo
---

![](/images/tree_growth.jpeg)


```R

library(tidyverse)
library(networkD3)


iris_clusters <- iris %>%
  select(Sepal.Length:Petal.Width) %>%
  dist() %>%
  hclust() %>%
  as.radialNetwork()

radialNetwork(iris_clusters, 
              # a few small style hacks
              fontSize = 0,
              nodeStroke = "#FFF",
              linkColour = "#000", 
              opacity = .1)

```


<iframe width='100%' height='400px' frameBorder='0' src='http://etachov.github.io/polar_dendrogram.html'></iframe>