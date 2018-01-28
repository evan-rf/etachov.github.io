---
layout:     post
title:      New York Times Op-Ed Link Networks
date:       2018-01-28
summary:    
categories: projects
---

![](/images/2018-01-28-nyt-graph-header.png)

When looking at communities of writers and thinkers, a fundamental question is who is talking about whom. For academics, we look at [citiations networks](https://kieranhealy.org/blog/archives/2013/06/18/a-co-citation-network-for-philosophy/). For bloggers and social media personalities, we look at [hyperlink](http://www.tandfonline.com/doi/abs/10.1080/13183222.2008.11008970) or [follower](https://www.aaai.org/ocs/index.php/ICWSM/ICWSM10/paper/viewFile/1468/1896) networks.

I applied this framework to another community of writers and thinkers: _The New York Times_ Op-Ed columnists. To do this, I collected the ten most recent articles by each of the eleven writers. I then extracted all of the hyperlinks from the articles to create the interactive graph below (full screen version [here](http://etachov.io/projects/nyt_citation_graph.html)). The data as well as the code I used to collect and analyze the links are available [on github](https://github.com/etachov/nyt_opinion_citations).  

<iframe width='100%' height='700px' frameBorder='0' src='http://etachov.io/projects/nyt_citation_graph_simple.html'></iframe>
<center><i>Explore the full screen graph  <a href = "http://etachov.io/projects/nyt_citation_graph.html" target = "_blank">here</a></i></center><br>

Looking at this graph a few things popped out for me. **First**, it's clear which outlets are at the center of the conversation. _The New York Times_ was (not surprisingly) cited by the all eleven columnists, _Washington Post_ by nine, _The Atlantic_ by seven, and CNN by seven. Taken together, these outlets are the "mainstream media", at least from the perspective of the _Times_' Op-Ed columnists.

![](/images/2018-01-28-nyt-graph-1-top-sources.gif)  

**Second**, startup media outlets are in the periphery of the network. Vox was cited by four of the columnists, while Business Insider, Buzzfeed, and Axios were cited by one columnist each. It will be interesting to see if any of these outlets shift into the center of the network over time.

![](/images/2018-01-28-nyt-graph-2-startup-media.gif)  

**Third**, there are clear differences in how the columnists approach linking. Nicholas Kristof linked out the most (92 links) and to the widest range of sources (51 different sites). Maureen Dowd took a different approach, linking out only 17 times to 8 different sources.

![](/images/2018-01-28-citation_frequency.png)  

**Finally**, Nicholas Kristof's subnetwork is the most distinct. Of the sites he linked to only 16% were also linked to other columnists. Conversely, Roger Cohen's subnetwork is the most embedded in the graph; 71% of his linked sites were also linked to by other columnists.

![](/images/2018-01-28-nyt-graph-4-distinct.gif)  

<br><center><b>Explore the full graph <a href = "http://etachov.io/projects/nyt_citation_graph.html" target = "_blank">here</a></b></center><br>

Keep in mind that we're looking at a limited and time-bound sample of articles. When I get a bit of time to figure out Selelium, I'm hoping to collect a year's worth of data. More to come!