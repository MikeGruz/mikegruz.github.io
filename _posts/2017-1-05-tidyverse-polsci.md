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
hearings = fread("http://comparativeagendas.s3.amazonaws.com/datasetfiles/congressional_hearings.csv") %>%
  as_data_frame()

# Pulling the Policy Agendas dataset on Gallup's Most Important Problem
mip = fread("http://comparativeagendas.s3.amazonaws.com/datasetfiles/Gallups_Most_Important_Problem.csv") %>%
  as_data_frame()
```

You might notice that I "piped" the data through `as_data_frame` to evaluate each of these datasets; this command turns these datasets into "tibbles" (another *tidyverse* term) that are more convenient than *R's* traditional `data.frame` structure. 

This is borne out by investigating the `mip` dataset; because it's a `data_frame`, the default output when viewed interactively is to show the first 10 observations:


```r
mip
```

```
## # A tibble: 1,470 × 5
##       id  year  percent majortopic Congress
##    <int> <int>    <dbl>      <int>    <int>
## 1      1  1946       NA          1       79
## 2      2  1947 0.347958          1       80
## 3      3  1948 0.221154          1       80
## 4      4  1949 0.288126          1       81
## 5      5  1950 0.233645          1       81
## 6      6  1951 0.228571          1       82
## 7      7  1952 0.214286          1       82
## 8      8  1953       NA          1       83
## 9      9  1954 0.333333          1       83
## 10    10  1955       NA          1       84
## # ... with 1,460 more rows
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
## # A tibble: 96,283 × 33
##       id     source CISYear Unpublished Chamber filter_House filter_Senate
##    <int>      <chr>   <chr>       <int>   <int>        <int>         <int>
## 1      1  93-H161-1    1993           0       1            1             0
## 2      2  93-H161-2    1993           0       1            1             0
## 3      3  93-H161-3    1993           0       1            1             0
## 4      4  93-H161-4    1993           0       1            1             0
## 5      5  93-H161-5    1993           0       1            1             0
## 6      6  93-H161-6    1993           0       1            1             0
## 7      7  93-H161-7    1993           0       1            1             0
## 8      8  93-H161-8    1993           0       1            1             0
## 9      9  93-H161-9    1993           0       1            1             0
## 10    10 93-H161-10    1993           0       1            1             0
## # ... with 96,273 more rows, and 26 more variables: filter_Joint <int>,
## #   CISCommittee <chr>, Sequence <chr>, Month <int>, Congress <int>,
## #   year <int>, FiscalYear <int>, Days <int>, Sessions <int>,
## #   Committee1 <int>, Subcommittee1 <int>, Committee2 <int>,
## #   Subcommittee2 <int>, filter_Referral <int>, filter_Approp <int>,
## #   filter_Agency <int>, filter_Program <int>, filter_Admin <int>,
## #   description <chr>, USMajorTopic <int>, USsubTopicCode <int>,
## #   majortopic <int>, subtopic <int>, ReferralBills <chr>,
## #   StartDate <chr>, AdditionalDates <chr>
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

![plot of chunk econplot](/figure/source/2017-1-05-tidyverse-polsci/econplot-1.svg)

..while the public has historically tended to expend more attention on Civil Rights issues than Congress (`majortopic == 2`):


```r
ggplot(data = filter(cong.mip, majortopic == 2), 
       aes(x = cong.perc, y = mip.perc)) +
  geom_point() + theme_minimal() + 
  ggtitle("Public and Congressional Attention to Civil Rights Issues")
```

![plot of chunk crplot](/figure/source/2017-1-05-tidyverse-polsci/crplot-1.svg)














