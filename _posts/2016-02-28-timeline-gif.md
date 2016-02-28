---
layout:     post
title:      Election Timeline Gif
date:       2016-02-28
summary:    Creating timeline gifs using ggplot2, gganimate and ggrepel
categories: viz
---

![](/images/upcoming_elections.gif)

Using <a href = "http://etachov.github.io/munging/2016/02/27/election-feed/" target = "_blank">yesterday's election data</a> from IFES I made a timeline gif using two cool tools from the ggplot2 <a href = "http://www.ggplot2-exts.org/" target = "_blank">extension universe</a>: <a href = "https://github.com/dgrtwo/gganimate" target = "_blank">gganimate</a> (makes change-over-time gifs a stanp) and <a href = "https://github.com/slowkow/ggrepel" target = "_blank">ggrepel</a> (smart placement for labels).

A gist of the code is below: 

<script src="https://gist.github.com/etachov/e5f97792f846023bfba8.js"></script>