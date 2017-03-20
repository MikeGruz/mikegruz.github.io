---
title: "Mapping in R"
date: 2017-03-16
comments: true
layout: post
share: true
output: html_document
---





Mapping used to be one of those things that always seemed a bit out of reach to all but the GIS folks. Let's be honest: *really good* mapping still very much is. But it like so many things in data manipulation and visualization, it has gotten quite a bit easier to create decent map-based visualizations in *R*.
<!--more-->
*R* has had decent mapping capabilities for some time, but (like so many things) these have gotten more accessible with Hadley Wickham's *ggplot* package, which includes the *map_data* set of shapes for political boundaries.

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
  geom_polygon(aes(x=long, y=lat, group=group), colour="black", fill='white', size=.25) +
  coord_fixed(1.3)
```

![plot of chunk stateplot](/figure/source/2017-03-16-mapping-in-R/stateplot-1.svg)

Easy-peasy. Let's try the same for counties.


```r
# pull in county boundaries data
counties <- map_data("county")

ggplot(data=counties) + 
  geom_polygon(aes(x=long, y=lat, group=group), colour="black", fill='white', size=.25) +
  coord_fixed(1.3)
```

![plot of chunk countyplot](/figure/source/2017-03-16-mapping-in-R/countyplot-1.svg)
This is cool, but even cooler would be including some county-level data representing 2016 election returns. Luckily for us, [Tom McGovern](https://github.com/tonmcg) has provided [county-level election results for the 2012-2016 U.S. presidential elections](https://github.com/tonmcg/County_Level_Election_Results_12-16). Let's pull that data into *R* and combine it with the county boundaries.


```r
# load the RCurl package to download data from github, as CSV format
require(RCurl)

elect.data <- read_csv(getURL("https://raw.githubusercontent.com/tonmcg/County_Level_Election_Results_12-16/master/2016_US_County_Level_Presidential_Results.csv"))
```

```
## Warning: Missing column names filled in: 'X1' [1]
```

```r
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
Unfortunately for us, the counties map data has (lowercase) full names for the states, whereas the county-level election data has state abbreviations.

We can rectify this by changing the election data's state abbreviations to lowercase full names using the `tolower()` function and the `state.name` and `state.abbr` datasets included in the *R* `datasets` package. County names are also lowercase in the map data, so we'll fix that here as well.


```r
elect.data <- elect.data %>%
  # use match to match up state name with state abbreviation, then convert to lower
  # also, to make things easier, name this region, a la county map data
  mutate(region = tolower(state.name[match(state_abbr, state.abb)]),
         subregion = tolower(county_name))
```

First order of business here is to merge the county map data with the election results data. I tend to do a merge immediately and check the result to see if anything doesn't match up, which is very typical when using two (or more) data sources.

For those purposes, I'm going to use `anti_join` from the `dplyr` package, which returns only those rows in *x* (the first dataframe) that do not have matching values in *y* (the second dataframe).

We'll perform the join using state (*region* in both datasets) and county (*subregion* in county map data, *county_name* in election data).


```r
anti_join(counties, elect.data, by=c("region", "subregion")) %>%
  # collapse by region-county to see how many, if any, didn't merge
  mutate(x=1) %>%
  group_by(region, subregion) %>%
  summarise(sum(x))
```

```
## Source: local data frame [3,074 x 3]
## Groups: region [?]
## 
##     region subregion `sum(x)`
##      <chr>     <chr>    <dbl>
## 1  alabama   autauga       51
## 2  alabama   baldwin      116
## 3  alabama   barbour       51
## 4  alabama      bibb       40
## 5  alabama    blount       69
## 6  alabama   bullock       44
## 7  alabama    butler       18
## 8  alabama   calhoun       73
## 9  alabama  chambers       26
## 10 alabama  cherokee       45
## # ... with 3,064 more rows
```

So it turns out that 3,074 of our counties in the mapping data didn't join with anything in the election data. To put that in context, [there are 3,142 counties and county-equivalents in the United States](https://en.wikipedia.org/wiki/List_of_counties_by_U.S._state), meaning that we successfully merged a whopping 2.2% of our data. Sad!

It turns out that most of the county names in our election data have "County" or "Parish" appended to them, while the mapping data omits those designations, which prevents most of the joins of finding a match between the two. Let's go ahead and use *R's* core `sub` function to remove them.


```r
elect.data <- elect.data %>%
  # we've aleady made subregion lowercase
  mutate(subregion = sub(" county| parish", "", subregion))

# try the antijoin again, and check our count of leftover counties
anti_join(counties, elect.data, by=c("region","subregion")) %>%
  mutate(x=1) %>%
  group_by(region, subregion) %>%
  summarise(sum(x))
```

```
## Source: local data frame [51 x 3]
## Groups: region [?]
## 
##                  region  subregion `sum(x)`
##                   <chr>      <chr>    <dbl>
## 1               alabama    de kalb       41
## 2               alabama   st clair       94
## 3              arkansas st francis       14
## 4  district of columbia washington       10
## 5               florida    de soto       11
## 6               florida   st johns       40
## 7               florida   st lucie       12
## 8               georgia    de kalb       23
## 9              illinois    de kalb       13
## 10             illinois    du page       10
## # ... with 41 more rows
```

Progress! We only have 51 orphaned counties now. 

At this point, it looks like we might have punctuation issues in the county names, as the county data omits punctuation while the election data does not.


```r
elect.data %>%
  filter(grepl("\\.|'", subregion)) %>%
  select(region, subregion)
```

```
## # A tibble: 30 × 2
##       region   subregion
##        <chr>       <chr>
## 1    alabama   st. clair
## 2   arkansas st. francis
## 3    florida   st. johns
## 4    florida   st. lucie
## 5       iowa     o'brien
## 6   illinois   st. clair
## 7    indiana  st. joseph
## 8  louisiana st. bernard
## 9  louisiana st. charles
## 10 louisiana  st. helena
## # ... with 20 more rows
```


```r
counties %>%
  filter(grepl("\\.|'", subregion)) %>%
  select(region, subregion)
```

```
## [1] region    subregion
## <0 rows> (or 0-length row.names)
```

Again, we'll use `sub` to remove that punctuation.


```r
elect.data <- elect.data %>%
  # use gsub to remove all instances of punctuation, making sure to
  # "escape" the period (\\.), which is a special character
  mutate(subregion = gsub("\\.|'", "", subregion))

# check the antijoin
anti_join(counties, elect.data, by=c("region","subregion")) %>%
  mutate(x = 1) %>%
  group_by(region, subregion) %>%
  summarise(sum(x))
```

```
## Source: local data frame [21 x 3]
## Groups: region [?]
## 
##                  region  subregion `sum(x)`
##                   <chr>      <chr>    <dbl>
## 1               alabama    de kalb       41
## 2  district of columbia washington       10
## 3               florida    de soto       11
## 4               georgia    de kalb       23
## 5              illinois    de kalb       13
## 6              illinois    du page       10
## 7              illinois   la salle       13
## 8               indiana    de kalb        6
## 9               indiana   la porte       22
## 10          mississippi    de soto       24
## # ... with 11 more rows
```

Almost there. On further investigation, it looks like county names like "DeKalb" in Indiana have a space where there shouldn't be one in the counties ("De Kalb") data. Let's change the election data to include that space (while I go and make a pull request in the ggplot library). We will match all of the strings manually using an `ifelse` statement, and then arbitrarily enter a space after the second character using `sub`.


```r
# we'll match on this grep string
county.matches <- "desoto|dekalb|dupage|lasalle|laporte|lamoure|dewitt"

elect.data <- elect.data %>% 
  mutate(subregion = ifelse(grepl(county.matches, subregion),
                 # if county matches, place space at third character
                 sub("^([a-z]{2})([a-z]+)", "\\1 \\2", subregion),
                 # if no match, keep the same
                 subregion)
  )
```

Let's try the join again.


```r
anti_join(counties, elect.data, by=c("region","subregion")) %>%
  mutate(x=1) %>%
  group_by(region, subregion) %>%
  summarise(sum(x))
```

```
## Source: local data frame [8 x 3]
## Groups: region [?]
## 
##                 region            subregion `sum(x)`
##                  <chr>                <chr>    <dbl>
## 1 district of columbia           washington       10
## 2              montana yellowstone national       30
## 3         south dakota              shannon       23
## 4             virginia              hampton       24
## 5             virginia         newport news       25
## 6             virginia              norfolk       40
## 7             virginia              suffolk       19
## 8             virginia       virginia beach       64
```

Close enough! (Change this later). Let's go ahead and join the two datasets.


```r
counties.elect <- full_join(counties, elect.data, by=c("region","subregion"))
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
  geom_polygon(aes(x=long, y=lat, group=group, fill=trump.perc)) +
  # use scale_fill_gradient to do a red-blue gradient
  scale_fill_gradient2(low = "blue", mid="grey", high = "red", name="Trump Win %") +
  coord_fixed(1.3) + theme_minimal() +
  theme(axis.title = element_blank(), 
        axis.text =element_blank(),
        panel.grid = element_blank())
```

![plot of chunk county.elect.map](/figure/source/2017-03-16-mapping-in-R/county.elect.map-1.svg)
# Plotting USDA Funding Data Against County Voting Behavior

Forthcoming..


```r
# get usda funding data
# usda <- getURL("http://download.usaspending.gov/datadownlods/Assistance/a3bc5309d872/Data_Feed.csv") %>%
#   read_csv()
# 
# usda.funds <- usda %>%
#   mutate(region = tolower(state.name[match(recipient_state_code, state.abb)]),
#          subregion = tolower(recipient_county_name)) %>%
#   group_by(region, subregion) %>%
#   summarise(funding = sum(fed_funding_amount, na.rm=TRUE))
# 
# counties.usda <- full_join(counties, usda.funds, by=c("region","subregion"))
# 
# ggplot(data=counties.usda) + geom_polygon(aes(x=long, y=lat, group=group, fill=funding)) +
#   coord_fixed(1.3) + theme_nothing()
```






