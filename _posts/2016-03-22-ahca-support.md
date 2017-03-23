---
title: "Tracking House Republicans' Support for the AHCA"
date: 2017-03-22
comments: true
layout: post
share: true
output: html_document
---










With the House vote on the *American Health Care Act* (AHCA) nearly upon us, a great deal of attention has been rightly given to the razor-thin margins President Trump and Republican leadership has in pushing this bill through Congress. As the [New York Times explained this week](https://www.nytimes.com/interactive/2017/03/20/us/politics/health-care-whip-count.html), the bill needs 216 Republican votes in the House to pass (no Democrats support the AHCA). 

According to the *NYT*, As of Thursday, March 23, there are 149 likely Republican supporters of the bill, while 44 are undecided, 15 have concerns or lean no, and 29 are likely no-votes. This bill cannot pass with 29 "no" votes.

Being so close to the election of Donald Trump as President of the United States, combined with the fact that House incumbents' next election is *always* looming, it can be useful to examine whip counts in relation to Trump's 2016 electoral performance in each Congressional district.
<!--more-->
To that end, I've pulled district-level electoral outcomes from [Daily Kos](http://www.dailykos.com/story/2012/11/19/1163009/-Daily-Kos-Elections-presidential-results-by-congressional-district-for-the-2012-2008-elections), along with the [*New York Times'* ongoing whip count of support and opposition among House Republicans](https://www.nytimes.com/interactive/2017/03/20/us/politics/health-care-whip-count.html).

In each of the following plots, the differential between Trump and Clinton's win percentage is plotted, with values at *0* indicating a 50-50 split between the candidates and higher values indicating a greater win differential for Trump (plotted in bright red for highest values).

![plot of chunk support.map](/figure/source/2016-03-22-ahca-support/support.map-1.svg)

Unsurprisingly, many of the 136 House Republicans who have come out in support for the AHCA hail from districts that Trump won handily. In fact, a full 120 (80.5%) of the districts of House Republicans supportive of the AHCA were won by over 10% by Trump; 79 (53%) of them were won by a margin of 20% or more by Trump. 

![plot of chunk undecided.map](/figure/source/2016-03-22-ahca-support/undecided.map-1.svg)

As one can imagine, it is the House Republicans sitting on the fence (above) and those who have concerns or are outright in opposition (below) where it gets more interesting. 

First off, 31 (70.5%) of the fence-sitting House Republicans hail from districts that Trump won by 10 points or more. Moreover, 11 (25%) of undecideds reside in districts that Trump won by *5 points or less*.

With the beginnings of the 2018 midterms already in gear, along with [influential interest groups running ads against the AHCA](http://www.cbsnews.com/news/major-health-advocacy-groups-announce-opposition-to-gop-plan-to-replace-obamacare/), these House Republicans in particular are in a bit of a bind.

![plot of chunk lean.map](/figure/source/2016-03-22-ahca-support/lean.map-1.svg)

Since yesterday, the NYT Whip Count has been updated to include those with concerns/those leaning no, so these individuals are mapped above. Trump won 11 (73.3%) of these districts by 10 points or more, and two by 5 points or fewer (13.3%).

![plot of chunk opponent.map](/figure/source/2016-03-22-ahca-support/opponent.map-1.svg)

Looking to those House Republicans who have come out either in opposition to the AHCA or voicing concerns about the legislation, the picture becomes even more bleak (there is *not* much dark red on the map above). 

To be sure, for those House Republicans voicing concerns or outright opposition to the bill, 17 (58.6%) hail from districts that Trump won by 10% or more, and 7 (24%) come from districts that Trump won by 5 points or under. 

As said earlier, this bill can only pass with 21 "no" votes on the Republican side of the aisle, so this is going to be a close one.

Even pulling in all members from districts Trump won by 5 points or more (42 undecideds and 37 opposed) brings the total to 189, still short of passage.

Should be interesting to watch over the next two days.

A few additional illustrations are included below, first of which is the mean Trump win percentage in each category of AHCA support, and second of which is an illustration of only those districts of AHCA supporters that Trump lost in 2016.


```
## 
## -----------------------------------
##  support_oppose     mean      sd   
## ----------------- -------- --------
##      Support      29.53143 15.53274
## 
## Undecided/Unclear 22.84144 17.43685
## 
## Lean No/Concerns  21.73175 14.57289
## 
##        No         21.84383 13.37025
## -----------------------------------
```

```
## 
## ----------------------------------------------------
##  support_oppose     mean      sd        se       n  
## ----------------- -------- -------- ---------- -----
## Lean No/Concerns  21.73175 14.57289 0.27728972 2762 
## 
##        No         21.84383 13.37025 0.14563886 8428 
## 
##      Support      29.53143 15.53274 0.08690673 31944
## 
## Undecided/Unclear 22.84144 17.43685 0.16369205 11347
## ----------------------------------------------------
```

![plot of chunk trump.lose](/figure/source/2016-03-22-ahca-support/trump.lose-1.svg)


```
## Error in eval(expr, envir, enclos): object 'ahca.trump' not found
```
