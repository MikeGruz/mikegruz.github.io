---
title: "Tracking House Republicans' Support for the AHCA"
date: 2017-03-22
comments: true
layout: post
share: true
output: html_document
---








With the House vote on the *American Health Care Act* (AHCA) nearly upon us, a great deal of attention has been rightly given to the razor-thin margins President Trump and Republican leadership has in pushing this bill through Congress. As the [New York Times explained this week](https://www.nytimes.com/interactive/2017/03/20/us/politics/health-care-whip-count.html), the bill needs 216 Republican votes in the House to pass (no Democrats support the AHCA). 

As of Wednesday, March 22, there are only 136 Republican supporters of the bill, while 56 are on the fence and another 45 have concerns or oppose the bill. This means that passage in the House will count on all of the fence-sitters to give a yea vote, plus 24 of those opposed or voicing concerns.

Being so close to the election of Donald Trump as President of the United States, combined with the fact that House incumbents' next election is *always* looming, it can be useful to examine whip counts in relation to Trump's 2016 electoral performance in each Congressional district.
<!--more-->
To that end, I've pulled district-level electoral outcomes from [Daily Kos](http://www.dailykos.com/story/2012/11/19/1163009/-Daily-Kos-Elections-presidential-results-by-congressional-district-for-the-2012-2008-elections), along with the [*New York Times'* ongoing whip count of support and opposition among House Republicans](https://www.nytimes.com/interactive/2017/03/20/us/politics/health-care-whip-count.html).

In each of the following plots, the differential between Trump and Clinton's win percentage is plotted, with values at *0* indicating a 50-50 split between the candidates and higher values indicating a greater win differential for Trump (plotted in bright red for highest values).

![plot of chunk support.map](/figure/source/2016-03-22-ahca-support/support.map-1.svg)

Unsurprisingly, many of the 136 House Republicans who have come out in support for the AHCA hail from districts that Trump won handily. In fact, a full 110 (81%) of the districts of House Republicans supportive of the AHCA were won by over 10% by Trump; 71 (52%) of them were won by a margin of 20% or more by Trump. 

![plot of chunk undecided.map](/figure/source/2016-03-22-ahca-support/undecided.map-1.svg)

As one can imagine, it is the House Republicans sitting on the fence (above) and those who have concerns or are outright in opposition (below) where it gets more interesting. 

First off, 40 (71%) of the fence-sitting House Republicans hail from districts that Trump won by 10 points or more. Moreover, 14 (25%) of undecideds reside in districts that Trump won by *5 points or less*.

With the beginnings of the 2018 midterms already in gear, along with [influential interest groups running ads against the AHCA](http://www.cbsnews.com/news/major-health-advocacy-groups-announce-opposition-to-gop-plan-to-replace-obamacare/), these House Republicans in particular are in a bit of a bind.

![plot of chunk opponent.map](/figure/source/2016-03-22-ahca-support/opponent.map-1.svg)

Looking to those House Republicans who have come out either in opposition to the AHCA or voicing concerns about the legislation, the picture becomes even more bleak (there is *not* much dark red on the map above). 

To be sure, for those House Republicans voicing concerns or outright opposition to the bill, 29 (64%) hail from districts that Trump won by 10% or more, and only 8 (18%) come from districts that Trump won by 5 points or under. 

Let's assume that the Republican Leadership whips up its members who come from districts that Trump handily won by a margin of 10% or more: that's 40 undecideds and 29 who currently oppose, bringing the vote tally to 179, far short of the 216 necessary for passage. 

Even pulling in all members from districts Trump won by 5 points or more (42 undecideds and 37 opposed) brings the total to 189, still short of passage.

Should be interesting to watch over the next two days.

![plot of chunk trump.bar](/figure/source/2016-03-22-ahca-support/trump.bar-1.svg)






