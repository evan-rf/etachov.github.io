---
layout:     post
title:      What Sites do New York Times Opinion Columnists Cite?
date:       2018-01-28
summary:    
categories: projects
---

![](/images/2018-01-28-nyt-graph-header.png)

When looking at communities of writers and thinkers, the most basic question is who is talking about whom? For scientists, the question is framed in terms of citiations networks. For bloggers, we look at which blogs link to other blogs. 

I decided to apply this framework to another community of writers and thinkers: the  columnists of _The New York Times_ Opinion page. To do this, I collected the ten most recent articles from all 11 columnists in residence. I then extracted all of the hyperlinks from the body of the articles to create the interactive graph below. The code I used to collect, analyze, and visualize the links is available [on github](https://github.com/etachov/nyt_opinion_citations).

<iframe width='100%' height='800px' frameBorder='0' src='http://etachov.io/projects/nyt_citation_graph_simple.html'></iframe>
*_Explore the full screen graph  [here](http://etachov.io/projects/nyt_citation_graph.html)_*

Looking at this graph a few things popped out for me. First, it's clear which outlets are at the center of the conversation. _The New York Times_ was cited by the all 11 of the columnists, _Washington Post_ by nine, _The Atlantic_ by seven, and CNN by seven. Taken together, these sources represent mainstream sources, at least from the perspetive of the Times' Opinion columnists.

![](/images/2018-01-28-nyt-graph-1-top-sources.gif)

Second, startup media outlets are in the periphery of the network. Vox was cited by four of the columists, while Business Insider, Buzzfeed, and Axios were cited by one columnist each. It will be interesting to see which of these outlets shift into the center of the network over time.


![](/images/2018-01-28-nyt-graph-2-startup-media.gif)

Third, there are clear differences in how the columnists approach linking. Nicholas Kristof links out the most (92 links) and to the widest range of sources (51 different sites). Maureen Dowd takes a different approach, linking out only 17 times to 8 different sources.

![](/images/2018-01-28-citation_frequency.png)

Finally, Nicholas Kristoff's subnetwork is the most distinct. Of the sites he linked to only 16% were also linked to by another columnist. Conversely, Roger Cohen's subnetwork is the most embedded in the graph; 71% of his linked sites were also linked to by other columnists.

![](/images/2018-01-28-nyt-graph-4-distinct.gif)

More to come!