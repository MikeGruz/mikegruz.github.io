---
title: "Agenda-Setting Between the U.S. Congress and Public"
output: html_document
date: 2017-1-05
comments: true
layout: post
share: true
---



The combination of the Hadley Wickham *tidyverse* and Stefan Milton's *magrittr* (also a part of the tidyverse) has made life *so much easier* for people who spend a great deal of time munging data - which is to say, pretty much anyone who does any amount of data munging. 
<!--more-->
In the spirit of their contributions to those of us who spend inordinant amounts of time doing just that, I wanted to give back by showing the use of these tools to whip up some analyses of some political science datasets. To that end, I'm going to make use of Frank Baumgartner and Bryan Jones' *Policy Agendas Project* dataset.

We'll start by importing the requisite packages, *tidyverse* (which includes *ggplot2*, *dplyr*, and *tidyr*, among other libraries), and *data.table*, which is useful for direct import of online data sources.


```r
require(tidyverse)
require(data.table)
```

Next, we'll download my dataset on control of Congress/Presidency, as well as the Policy Agendas Project datasets inclusive of Congressional hearings on policy topics and Gallup's "Most Important Problem" dataset.


```r
# Pulling the Policy Agendas dataset on Congressional hearings
hearings = fread("http://comparativeagendas.s3.amazonaws.com/datasetfiles/congressional_hearings.csv")

# Pulling the Policy Agendas dataset on Gallup's Most Important Problem
mip = fread("http://comparativeagendas.s3.amazonaws.com/datasetfiles/Gallups_Most_Important_Problem.csv")
```

You might notice that I used `as_data_frame` to evaluate each of these datasets; this command turns these datasets into "tibbles" (another *tidyverse* term) that are more convenient than *R's* traditional `data.frame` structure. 

This is borne out by investigating the `mip` dataset; because it's a `data_frame`, the default output when viewed interactively is to show the first 10 observations:


```r
mip
```

```
##         id year    percent majortopic Congress
##    1:    1 1946         NA          1       79
##    2:    2 1947 0.34795800          1       80
##    3:    3 1948 0.22115400          1       80
##    4:    4 1949 0.28812600          1       81
##    5:    5 1950 0.23364500          1       81
##   ---                                         
## 1466: 1466 2011 0.05236300         25      112
## 1467: 1467 2012 0.06049300         25      113
## 1468: 1468 2013 0.10336827         25      113
## 1469: 1469 2014 0.08388879         25      113
## 1470: 1470 2015 0.08220624         25      114
```

This is pretty easy data to work with: basically we have a dataset made up of the year, policy topic in the *Policy Agendas Project* coding scheme (*majortopic*), percentage of survey respondents who said each topic was the most important problem facing the nation in that year (*mip.perc*), and the Congressional session for that year.

For purposes of distinguishing between public and Congressional attention later on (when we merge the two datasets), I'm going to rename `percent` to `mip.perc` and keep only that variable, the topic code (`majortopic`) and the year (`year`). I'll cover exactly how I'm using `dplyr` and its piping function (`%>%`) further below.


```r
mip = mip %>%
  select(year, majortopic, mip.perc = percent)
```

Things get a bit more complicated when we look at the Congressional hearings dataset, as this dataset is structured such that each Congressional hearing occupies a row, which means that if we want to compare Congressional hearings by year to "MIP" responses by year, we'll have to do some data munging.


```r
hearings
```

```
##            id       source CISYear Unpublished Chamber filter_House
##     1:      1    93-H161-1    1993           0       1            1
##     2:      2    93-H161-2    1993           0       1            1
##     3:      3    93-H161-3    1993           0       1            1
##     4:      4    93-H161-4    1993           0       1            1
##     5:      5    93-H161-5    1993           0       1            1
##    ---                                                             
## 96279: 103354 2014-H181-22    2014           0       1            1
## 96280: 103355 2014-H181-23    2014           0       1            1
## 96281: 103356 2014-H181-24    2014           0       1            1
## 96282: 103357 2014-H181-25    2014           0       1            1
## 96283: 103358 2014-H181-26    2014           0       1            1
##        filter_Senate filter_Joint CISCommittee Sequence Month Congress
##     1:             0            0          161        1    10      102
##     2:             0            0          161        2    11      102
##     3:             0            0          161        3    12      102
##     4:             0            0          161        4     7      102
##     5:             0            0          161        5     3      102
##    ---                                                                
## 96279:             0            0          181       22     1      113
## 96280:             0            0          181       23     1      113
## 96281:             0            0          181       24     3      113
## 96282:             0            0          181       25     2      113
## 96283:             0            0          181       26     3      113
##        year FiscalYear Days Sessions Committee1 Subcommittee1 Committee2
##     1: 1991       1992    1        5        102         10201        102
##     2: 1991       1992    1        4        102         10204          0
##     3: 1991       1992    4       11        102         10200          0
##     4: 1992       1992    1        2        102         10205          0
##     5: 1992       1992    1        0        102         10206        114
##    ---                                                                  
## 96279: 2014       2014    1        1        103         10307          0
## 96280: 2013       2013    1        1        103         10307          0
## 96281: 2013       2013    1        1        103         10307          0
## 96282: 2013       2013    4        4        103         10307          0
## 96283: 2013       2013    3        3        103         10306          0
##        Subcommittee2 filter_Referral filter_Approp filter_Agency
##     1:         10204               0             0             0
##     2:             0               0             0             0
##     3:             0               0             0             0
##     4:             0               0             0             0
##     5:         11403               0             0             0
##    ---                                                          
## 96279:             0               0             1             0
## 96280:             0               0             1             0
## 96281:             0               0             1             0
## 96282:             0               0             1             0
## 96283:             0               0             1             0
##        filter_Program filter_Admin
##     1:              0            0
##     2:              0            0
##     3:              0            0
##     4:              0            0
##     5:              0            0
##    ---                            
## 96279:              0            0
## 96280:              0            0
## 96281:              0            0
## 96282:              0            0
## 96283:              0            0
##                                                                                        description
##     1:                            GAO report, methodologies in nationwide food consumption survey.
##     2:         Investigate fatal fire at food products chicken processing plant in North Carolina.
##     3:                   Agricultural issues involved in the GATT multilateral trade negotiations.
##     4:                                                                           US honey program.
##     5: Conflicts over timber harvesting in Pacific Northwest habitats of the northern spotted owl.
##    ---                                                                                            
## 96279:                 No summary available, this is in the Consolidated Appropriations Act, 2014.
## 96280:                 No summary available, this is in the Consolidated Appropriations Act, 2014.
## 96281:                 No summary available, this is in the Consolidated Appropriations Act, 2014.
## 96282:                 No summary available, this is in the Consolidated Appropriations Act, 2014.
## 96283:                 No summary available, this is in the Consolidated Appropriations Act, 2014.
##        USMajorTopic USsubTopicCode majortopic subtopic ReferralBills
##     1:            4            499          4      499              
##     2:            5            501          5      501              
##     3:            4            401          4      401              
##     4:            4            402          4      402              
##     5:            7            709          7      709              
##    ---                                                              
## 96279:           20           2000         20     2000              
## 96280:           20           2000         20     2000              
## 96281:           20           2000         20     2000              
## 96282:           20           2000         20     2000              
## 96283:           20           2000         20     2000              
##        StartDate                 AdditionalDates
##     1:                                          
##     2:                                          
##     3:                                          
##     4:                                          
##     5:                                          
##    ---                                          
## 96279:    1/1/14                                
## 96280:    1/1/13                                
## 96281:   3/14/13                                
## 96282:   2/14/13 3/13/2013; 4/16/2013; 4/24/2013
## 96283:    3/5/13            3/14/2013; 3/19/2013
```

Basically, what we want is a dataset that is structured as **year-issue-hearing percentages**, so that we can directly compare the yearly "MIP" responses with the percentage of yearly Congressional hearings on each topic.

This is one of the many cases in which the `dplyr` package, which is included in the `tidyverse` package, really shines. Whereas just a few years ago one would have had to create multiple datasets to get this data into mergeable form, with `dplyr` we can do it in a few short steps:


```r
hearings.yearly = hearings %>%
  mutate(x = 1) %>%
  group_by(year, majortopic) %>%
  summarise(total = sum(x)) %>%
  mutate(cong.perc = total/sum(total)) %>%
  select(year, majortopic, cong.perc)

hearings.yearly
```

```
## Source: local data frame [1,384 x 3]
## Groups: year [69]
## 
##     year majortopic   cong.perc
##    <int>      <int>       <dbl>
## 1   1946          1 0.022193211
## 2   1946          2 0.007832898
## 3   1946          3 0.022193211
## 4   1946          4 0.039164491
## 5   1946          5 0.016971279
## 6   1946          6 0.009138381
## 7   1946          7 0.011749347
## 8   1946          8 0.028720627
## 9   1946          9 0.014360313
## 10  1946         10 0.054830287
## # ... with 1,374 more rows
```

There are more than a few things going on here, but two in particular stand out: 

1. The use of "piping" to pass the results of one command to another. This is denoted by `%>%`.  

2. While we have *technically* created five separate datasets in this block of code (every line after the initial `hearings.yearly = hearings %>%` assignment creates a new dataset), the piping functions merely pass each dataset on to the next function without committing those datasets to memory.

This is the glory of the *dplyr/tidyr* way of doing things - rather than creating a bunch of datasets en route to the final product, we can instead pass each step of the process into the next stage, thus drastically reducing the task of juggling multiple datasets that act only as precursors to what we ultimately want.

In the above code excerpt, we've done a few things.

First, we assigned a new dataset using an old dataset, which we piped:


```r
hearings.yearly = hearings %>%
```

Everything that we do in this code will get assigned to `hearings.yearly`.

Next, we add a vector of ones to the dataset, which will allow us to count the number of hearings within each year-issue using `mutate()`, which creates a new variable in the dataset:


```r
  mutate(x = 1) %>%
```

We next use `group_by()` to split the data by *year* and *majortopic* so that we can compute summary statistics within each year-topic:


```r
  group_by(year, majortopic) %>%
```

We then summarize (note the 's' in the function name). The `summarise()` function effectively collapses data, computing summary statistics. In this case, we are summarizing within `year` and `majortopic` (using `group_by()` in the previous line). We generate a new variable `total` by taking `sum(x)`.


```r
  summarise(total = sum(x)) %>%
```

Next, we generage a new variable `cong.perc` through `mutate()` by taking each row's total count (the count of hearings on each issue within each year) divided by the total number of hearings in each year:


```r
  mutate(cong.perc = total/sum(total)) %>%
```

Finally, we keep only those variables that we are interested in; in this case, `year`, the indicator for each issue topic `majortopic`, and the percentage of Congressional hearings held on each issue within each year, `cong.perc`.


```r
  select(year, majortopic, cong.perc)
```

The end result is `hearings.yearly`, a dataset that has exactly (but nothing more) what we needed:


```r
hearings.yearly
```

```
## Source: local data frame [1,384 x 3]
## Groups: year [69]
## 
##     year majortopic   cong.perc
##    <int>      <int>       <dbl>
## 1   1946          1 0.022193211
## 2   1946          2 0.007832898
## 3   1946          3 0.022193211
## 4   1946          4 0.039164491
## 5   1946          5 0.016971279
## 6   1946          6 0.009138381
## 7   1946          7 0.011749347
## 8   1946          8 0.028720627
## 9   1946          9 0.014360313
## 10  1946         10 0.054830287
## # ... with 1,374 more rows
```

Because we now have a dataset using the same structure as our "MIP" dataset, we can now merge the two (note that I am using `full_join` from the `tidyverse` package to complete the merge).


```r
cong.mip = full_join(hearings.yearly, mip, by=c('year','majortopic'))

cong.mip
```

```
## Source: local data frame [1,476 x 4]
## Groups: year [?]
## 
##     year majortopic   cong.perc mip.perc
##    <int>      <int>       <dbl>    <dbl>
## 1   1946          1 0.022193211       NA
## 2   1946          2 0.007832898       NA
## 3   1946          3 0.022193211       NA
## 4   1946          4 0.039164491       NA
## 5   1946          5 0.016971279       NA
## 6   1946          6 0.009138381       NA
## 7   1946          7 0.011749347       NA
## 8   1946          8 0.028720627       NA
## 9   1946          9 0.014360313       NA
## 10  1946         10 0.054830287       NA
## # ... with 1,466 more rows
```

At this point we can start taking a look at the public-Congress agendas to check for congruence (using `ggplot` to throw together some graphs). For example, Congressional and public attention to economic issues (`majortopic == 1`) is *very* strong:


```r
ggplot(data = filter(cong.mip, majortopic == 1), 
       aes(x = cong.perc, y = mip.perc)) +
  geom_point() + theme_minimal() + 
  ggtitle("Public and Congressional Attention to Economic Issues")
```

![plot of chunk econplot](/figure/./2017-1-05-tidyverse-polsci/econplot-1.svg)

..while the public has historically tended to expend more attention on Civil Rights issues than Congress (`majortopic == 2`):


```r
ggplot(data = filter(cong.mip, majortopic == 2), 
       aes(x = cong.perc, y = mip.perc)) +
  geom_point() + theme_minimal() + 
  ggtitle("Public and Congressional Attention to Civil Rights Issues")
```

![plot of chunk crplot](/figure/./2017-1-05-tidyverse-polsci/crplot-1.svg)














