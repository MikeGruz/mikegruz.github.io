---
title: "Mapping in R"
date: 2017-03-16
comments: true
layout: post
share: true
output: html_document
image:
  feature: https://mikegruz.github.io/figure/source/2017-03-16-mapping-in-R/unnamed-chunk-17-1.svg
---





Mapping used to be one of those things that always seemed a bit out of reach to all but the GIS folks. Let's be honest: *really good* mapping still very much is. But it like so many things in data manipulation and visualization, it has gotten quite a bit easier to create decent map-based visualizations in *R*.

*R* has had decent mapping capabilities for some time, but (like so many things) these have gotten more accessible with Hadley Wickham's *ggplot* package, which includes the *map_data* set of shapes for political boundaries.
<!--more-->
To quickly demonstrate how easy it is to create a map, we can access the shapefile data in *map_data* for state boundaries within the United States and quickly plot it.


```r
# load required libraries
require(ggplot2)

# pull state boundaries from map_data
states <- map_data("state")

# view data
head(states)
```

```
##        long      lat group order  region subregion
## 1 -87.46201 30.38968     1     1 alabama      <NA>
## 2 -87.48493 30.37249     1     2 alabama      <NA>
## 3 -87.52503 30.37249     1     3 alabama      <NA>
## 4 -87.53076 30.33239     1     4 alabama      <NA>
## 5 -87.57087 30.32665     1     5 alabama      <NA>
## 6 -87.58806 30.32665     1     6 alabama      <NA>
```

As you can see, dataframes from the *map_data* package include a row-column representation of the longitudes and latitudes of the chosen geography, along with the state (*region*) those boundaries represent.

We'll plot these regions as polygons (*geom_polygon*) using ggplot. Note that we'll use *coord_fixed(1.3)* to keep the aspect ratio consistent and correct.


```r
ggplot(data=states) + 
  geom_polygon(aes(x=long, y=lat, group=group), colour="black", fill='white', size=.2) +
  coord_fixed(1.3) + theme_void()
```

![plot of chunk stateplot](/figure/./2017-03-16-mapping-in-R/stateplot-1.svg)

Easy-peasy. Let's try the same for counties.


```r
# pull in county boundaries data
counties <- map_data("county")

ggplot(data=counties) + 
  geom_polygon(aes(x=long, y=lat, group=group), colour="black", fill='white', size=.2) +
  coord_fixed(1.3) + theme_void()
```

![plot of chunk countyplot](/figure/./2017-03-16-mapping-in-R/countyplot-1.svg)

This is cool, but even cooler would be including some county-level data representing 2016 election returns. Luckily for us, [Tom McGovern](https://github.com/tonmcg) has provided [county-level election results for the 2012-2016 U.S. presidential elections](https://github.com/tonmcg/County_Level_Election_Results_12-16). Let's pull that data into *R* and combine it with the county boundaries.


```r
# load the RCurl package to download data from github, as CSV format
require(RCurl)

elect.data <- read_csv(getURL("https://raw.githubusercontent.com/tonmcg/County_Level_Election_Results_12-16/master/2016_US_County_Level_Presidential_Results.csv"))

# inspect the data
head(elect.data)
```

```
## # A tibble: 6 × 11
##      X1 votes_dem votes_gop total_votes   per_dem per_gop  diff
##   <int>     <dbl>     <dbl>       <dbl>     <dbl>   <dbl> <dbl>
## 1     0     93003    130413      246588 0.3771595 0.52887 37410
## 2     1     93003    130413      246588 0.3771595 0.52887 37410
## 3     2     93003    130413      246588 0.3771595 0.52887 37410
## 4     3     93003    130413      246588 0.3771595 0.52887 37410
## 5     4     93003    130413      246588 0.3771595 0.52887 37410
## 6     5     93003    130413      246588 0.3771595 0.52887 37410
## # ... with 4 more variables: per_point_diff <chr>, state_abbr <chr>,
## #   county_name <chr>, combined_fips <int>
```

While McGovern's county-level election data has FIPS codes, our county mapping data does not (we could merge on county names, but this introduces *a lot* of headaches with naming schemes).

We'll make use of the `county.fips` data included with R to rectify this. This is what it looks like:


```r
head(county.fips)
```

```
##   fips        polyname
## 1 1001 alabama,autauga
## 2 1003 alabama,baldwin
## 3 1005 alabama,barbour
## 4 1007    alabama,bibb
## 5 1009  alabama,blount
## 6 1011 alabama,bullock
```

Let's go ahead and split `polyname` into `region` and `subregion` using the `separate()` function in `tidyr`.


```r
fips.codes <- county.fips %>%
  # create two new variables, splitting "polyname" on the comma
  separate(polyname, c("region", "subregion"), ",")

# inspect
head(fips.codes)
```

```
##   fips  region subregion
## 1 1001 alabama   autauga
## 2 1003 alabama   baldwin
## 3 1005 alabama   barbour
## 4 1007 alabama      bibb
## 5 1009 alabama    blount
## 6 1011 alabama   bullock
```

Now we'll merge in the county shape data to pull in the FIPS codes using `anti_join`, which keeps only those observations in the first dataset (`counties`) that don't match the second merged-in dataset (`fips.codes`).


```r
anti_join(counties, fips.codes, by=c("region","subregion")) %>%
  select(region,subregion) %>%
  group_by(region,subregion) %>%
  summarise(n())
```

```
## Source: local data frame [7 x 3]
## Groups: region [?]
## 
##           region subregion `n()`
##            <chr>     <chr> <int>
## 1        florida  okaloosa    37
## 2      louisiana st martin    85
## 3 north carolina currituck    97
## 4          texas galveston    62
## 5       virginia  accomack    83
## 6     washington    pierce    87
## 7     washington  san juan    53
```

Looks like we have seven problem counties that don't match up between them. Let's see what these are named in the `fips.codes` dataset.


```r
fips.codes %>%
  filter(grepl("okaloo|martin|curri|galve|acco|pierc|juan", subregion))
```

```
##     fips         region                subregion
## 1   8111       colorado                 san juan
## 2  12085        florida                   martin
## 3  12091        florida            okaloosa:main
## 4  12091        florida            okaloosa:spit
## 5  13229        georgia                   pierce
## 6  18101        indiana                   martin
## 7  21159       kentucky                   martin
## 8  22099      louisiana          st martin:north
## 9  22099      louisiana          st martin:south
## 10 27091      minnesota                   martin
## 11 31139       nebraska                   pierce
## 12 35045     new mexico                 san juan
## 13 37053 north carolina         currituck:knotts
## 14 37053 north carolina           currituck:main
## 15 37053 north carolina           currituck:spit
## 16 37117 north carolina                   martin
## 17 38069   north dakota                   pierce
## 18 48167          texas           galveston:main
## 19 48167          texas           galveston:spit
## 20 48317          texas                   martin
## 21 49037           utah                 san juan
## 22 51001       virginia            accomack:main
## 23 51001       virginia    accomack:chincoteague
## 24 53053     washington              pierce:main
## 25 53053     washington           pierce:penrose
## 26 53055     washington    san juan:lopez island
## 27 53055     washington    san juan:orcas island
## 28 53055     washington san juan:san juan island
## 29 55093      wisconsin                   pierce
```

So when we split out the region-subregion data using the comma (which separated region and subregion in the data) we apparently missed some sub-subregion splits (separated by a colon). This should be an easy fix: we'll just split that out again.


```r
fips.codes <- county.fips %>%
  # separate region and subregion
  separate(polyname, c("region","subregion"), ",") %>%
  # separate subregion and sub-subregion
  separate(subregion, c("subregion","subsubregion"), ":")
```

```
## Warning: Too few values at 3069 locations: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
## 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, ...
```

```r
# dplyr will complain about NAs, but it will work
```

The anti_join should now work:


```r
anti_join(counties, fips.codes, by=c("region","subregion")) %>%
  select(region,subregion) %>%
  group_by(region,subregion) %>%
  summarise(n())
```

```
## Source: local data frame [0 x 3]
## Groups: region [?]
## 
## # ... with 3 variables: region <chr>, subregion <chr>, n() <int>
```

Success - no non-merging data! Let's go ahead and perform a full join.


```r
counties <- full_join(counties, fips.codes, by=c("region","subregion"))
```

Because our county shape data now has FIPS codes tied to it, we can merge in the election data we pulled in earlier.

Like always, I'll do an `anti_join` to examine whether anything failed to merge.


```r
# check first to see if any non-joiners
anti_join(counties, elect.data, by=c("fips"="combined_fips")) %>%
  select(fips) %>%
  group_by(fips) %>%
  summarise(n())
```

```
## # A tibble: 0 × 2
## # ... with 2 variables: fips <int>, n() <int>
```

Looks good! Let's do the full merge.


```r
counties.elect <- full_join(counties, elect.data, by=c("fips"="combined_fips"))

head(counties.elect)
```

```
##        long      lat group order  region subregion fips subsubregion X1
## 1 -86.50517 32.34920     1     1 alabama   autauga 1001         <NA> 29
## 2 -86.53382 32.35493     1     2 alabama   autauga 1001         <NA> 29
## 3 -86.54527 32.36639     1     3 alabama   autauga 1001         <NA> 29
## 4 -86.55673 32.37785     1     4 alabama   autauga 1001         <NA> 29
## 5 -86.57966 32.38357     1     5 alabama   autauga 1001         <NA> 29
## 6 -86.59111 32.37785     1     6 alabama   autauga 1001         <NA> 29
##   votes_dem votes_gop total_votes   per_dem   per_gop  diff per_point_diff
## 1      5908     18110       24661 0.2395685 0.7343579 12202         49.48%
## 2      5908     18110       24661 0.2395685 0.7343579 12202         49.48%
## 3      5908     18110       24661 0.2395685 0.7343579 12202         49.48%
## 4      5908     18110       24661 0.2395685 0.7343579 12202         49.48%
## 5      5908     18110       24661 0.2395685 0.7343579 12202         49.48%
## 6      5908     18110       24661 0.2395685 0.7343579 12202         49.48%
##   state_abbr    county_name
## 1         AL Autauga County
## 2         AL Autauga County
## 3         AL Autauga County
## 4         AL Autauga County
## 5         AL Autauga County
## 6         AL Autauga County
```


We're about ready to map out the win percentage for Trump in 2016; we just need to create a variable subtracting Clinton's percentage of the vote from Trump's percentage.


```r
counties.elect <- counties.elect %>%
  mutate(trump.perc = (per_gop - per_dem)*100)
```

And now we'll map it, using the same `ggplot` code as before while adding a `fill=` statement to fill in a gradient based on the percent differential between the two candidates. Additionally, we'll remove the gridlines and axis labels from the figure.


```r
# we're going to use the "scales" package to mute the color

ggplot(data=counties.elect) + 
  # use "fill=trump.perc" to create a gradient based on two-party differential
  geom_polygon(aes(x=long, y=lat, group=group, fill=trump.perc), colour="black", size=.1) +
  # use scale_fill_gradient to do a red-blue gradient
  scale_fill_gradient2(low = "blue", mid="white", high = "red", name="Trump Win %") +
  coord_fixed(1.3) + theme_void()
```

![plot of chunk county.elect.map](/figure/./2017-03-16-mapping-in-R/county.elect.map-1.svg)


# Plotting Elite Polarization in US Congressional Districts

This is all well and good, but we frequently want to visualize data in a map format that isn't necessarily available in the `map_data` library. For example, Census tracts, Congressional districts, and state legislative districts might be a few we're interested in.

We can make use of the U.S. Census Bureau's shapefiles to get at quite a few mapping visualizations. To that end, I am going to pull down shapefiles for U.S. Congressional districts, convert them to a format R can use, and merge the resulting data with Poole & Rosenthal's *DW-Nominate* data to visualize party polarization in the 113th House of Representatives.

First, load the requisite packages. We'll make use of `rgdal`, `rgeos`, and `maptools` to convert the Census shapefiles to a useable format.


```r
require(rgdal)
require(maptools)
require(rgeos)
```

The Census shapefiles can be [found at this link](http://www2.census.gov/geo/tiger/GENZ2013/cb_2013_us_cd113_20m.zip).

We're going to first read in the shapefiles (using `readOGR`), pull in the ID for each Congressional district (using `row.names()`), transform the projection (`spTransform`), use `gBuffer` to deal with polygon issues in the shape paths, and finally use `fortify` to convert the shapes to a dataframe R can deal with.

Note that the use of GIS shapefiles is a rich and varied concept that a person (like me) has no business messing around with. Still, we'll proceed.

Note that I pulled the "aea" projection parameters from [Kris Eberwein's](https://github.com/keberwein) Github Gist on [Map Projections for the USA.](https://gist.github.com/keberwein/de42982e18e242804c97a728c0ad5535) Like I said, I have no business in the business of GIS.


```r
# read in the shapefiles from the directory, denoted by "dsn"
#
# note that the "layer" attribute is set to the base filename of the shapefiles 
districts <- readOGR(dsn = "./data/cb_2013_us_cd113_20m/",
                     layer = "cb_2013_us_cd113_20m",
                     stringsAsFactors = FALSE)
```

```
## OGR data source with driver: ESRI Shapefile 
## Source: "./data/cb_2013_us_cd113_20m/", layer: "cb_2013_us_cd113_20m"
## with 437 features
## It has 8 fields
## Integer64 fields read as strings:  ALAND AWATER
```

```r
# pull in row names for merging
districts@data$id <- row.names(districts@data)

# transform shapes to the projection we desire
districts.tran <- spTransform(districts, 
                            CRS("+proj=aea +lat_1=29.5 +lat_2=45.5 +lat_0=37.5 +lon_0=-96"))

# buffer the shapes
districts.points <- gBuffer(districts.tran, byid=TRUE, width=0)

# fortify the data into a dataframe
districts.df <- fortify(districts.points, region="id")

# merge in metadata from shapefiles
districts.df <- full_join(districts.df, districts@data, by="id")
```

Because I already know that the state codes in the Census data use *FIPS* codes, while Poole & Rosenthal's data uses *ICPSR* codes, I'll need to convert between the two. Using a conversion chart located [here](http://staff.washington.edu/glynn/StateFIPSicsprAB.xls), I'll just merge the icpsr codes on joins with the FIPS codes.

I'm also going to convert the state FIPS codes (`STATEFP`), as well as Congressional district codes (`CD113FP`), to numeric (they are read in as strings initially). Moreover, because the Census codes at-large districts as *0* while the DW-Nominate data codes those districts as *1*, I'll take care of that conversion here as well.


```r
# create dataframe of FIPS-ICPSR codes
fips.icpsr <- data_frame(
  fips = c(1,2,4,5,6,8,9,10,11,12,13,15,16,17,18,19,20,21,22,23,24,25,26,27,28,29,30,31,32,33,34,35,36,37,38,39,40,41,42,44,45,46,47,48,49,50,51,53,54,55,56),
  icpsr = c(41,81,61,42,71,62,1,11,55,43,44,82,63,21,22,31,32,51,45,2,52,3,23,33,46,34,64,35,65,4,12,66,13,47,36,24,53,72,14,5,48,37,54,49,67,6,40,73,56,25,68)
)


districts.df <- districts.df %>%
  # get state icpsr code
  mutate(cd = as.numeric(CD113FP),
         fips = as.numeric(STATEFP),
         # recode "0" districts as "1" (these are at-large districts)
         cd = ifelse(cd==0, 1, cd))

# merge in icpsr state codes, joining on fips codes
districts.df <- full_join(districts.df, fips.icpsr, by="fips") %>%
  # rename icpsr to state for merge with DW data
  rename(state = icpsr)
```

Pull in the DW-Nominate data from the web, and drop all but data for the 113th Congress, as well as any data from presidents and the Senate (both of which are coded as *0* in the DW data's `cd` variable).

I'm going to import the Stata version of the DW data using the `haven` package.


```r
# load in 113th Congress nominate file from the web
house.dw <- haven::read_dta("./data/HANDSL01113C20_BSSE.DTA") %>%
  # remove senate/president scores, keep just 113th
  filter(cong==113 & cd != 0) 
```

Now do a `full_join` to merge the district shapefiles with the DW-Nominate data. We'll drop Alaska, Hawaii, DC and Puerto Rico for now to make plotting simple.

While we're doing this, we'll also create a variable `polar`, which is the absolute value of the first ideological dimension `dwnom1`, so that we can plot absolute polarization.


```r
# merge dw and district map
districts.dw <- full_join(districts.df, house.dw, by=c("state","cd")) %>%
  # drop alaska (2), hawaii (15), DC (11) and PR (72) 
  filter(!(fips %in% c(2,15,11,72))) %>%
  # create absolute value of dwnom1 for polarization
  mutate(polar = abs(dwnom1))
```

At this point, plotting is pretty much identical to our county plots earlier. We'll create a `ggplot` using our `districts.dw` data, add a layer for polygons, with fills based on partisan polarization levels. Note that values below *0* indicate increasing levels of liberalism while values above *0* indicate higher values of conservatism.

I'm not a huge fan of mapping using "red state, blue state, purple state," because to me and my colorblindness, purple looks more blue than it probably should, so I'll use white as the midpoint for moderates in the House.


```r
# plot party extremes
ggplot(districts.dw) + 
  geom_polygon(aes(x=long, y=lat, group=group, fill=dwnom1), colour="black", size=.1) +
  scale_fill_gradient2(low="blue", mid="white", high="red", name="Party Polarization") +
  theme_void() + coord_fixed(1) + ggtitle("Party Polarization in the 113th U.S. Congress")
```

![plot of chunk unnamed-chunk-16](/figure/./2017-03-16-mapping-in-R/unnamed-chunk-16-1.svg)

Let's do the same for absolute polarization, using white for moderates and orange for polarized members of Congress.


```r
# plot ideological extremity without party
ggplot(districts.dw) + 
  geom_polygon(aes(x=long, y=lat, group=group, fill=polar), colour="black", size=.1) +
  scale_fill_gradient(low="white", high="orange", name="Ideological Polarization") +
  theme_void() + coord_fixed(1) + ggtitle("Absolute Polarization in the 113th U.S. Congress")
```

![plot of chunk unnamed-chunk-17](/figure/./2017-03-16-mapping-in-R/unnamed-chunk-17-1.svg)


