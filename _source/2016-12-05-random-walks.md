---
title: "Generating Random Walks in R"
output: html_document
date: 2016-12-05
tags: [R, data generation, random walk]
layout: post
comments: true
share: true
---



*Note: The code in this post was partially derived from an answer by [Jake Burkhead](http://stackoverflow.com/users/2317463/jake-burkhead) over at [Stack Overflow](http://stackoverflow.com/questions/21991130/simulating-a-random-walk/21991340#21991340)*

It can be useful for illustration purposes to be able to show basic concepts such as "random walks" using R. If you're not familiar with [random walks](https://en.wikipedia.org/wiki/Random_walk), the concept is usually applied to a Markov Chain process, wherein the current value of some variable is dependent upon only its previous value (not *values*, mind you), with deviations from the previous value being either `-1` or `1`. 
<!--more-->
In other words, given some random-walking variable `x` at time `t`, the next value will either be `x+1` or `x-1` at time `t+1`. While one of the most notable applications of this is to [movements of markets](https://en.wikipedia.org/wiki/Random_walk_hypothesis), it also has great applicability to other concepts, such as Markov Chain language generators, animal movement, etc.

Just for kicks, generating a random walk in R is simple. We simply generate a vector of numbers randomly sampled from a vector `(-1,1)`, and take the cumulative sum of that numerical vector.


```r
require(ggplot2)

set.seed(123) # for reproducibility
n = 500 # length of random walk

rand.walk = data.frame(
  value = cumsum(sample(c(-1,1), size=n, replace=TRUE)),
  n = 1:n
)

ggplot(rand.walk, aes(x=n, y=value)) + geom_line() + theme_minimal()
```

![plot of chunk randomwalk](/figure/./2016-12-05-random-walks/randomwalk-1.svg)

We can also do this in more than one dimension. Shown below is a two-dimensional random walk, which looks like a randomly-generated dungeon crawl:


```r
n = 5000

d2.rand.walk = data.frame(
  x = cumsum(sample(c(-1,1), size=n, replace=TRUE)),
  y = cumsum(sample(c(-1,1), size=n, replace=TRUE))
)


ggplot(d2.rand.walk, aes(x=x, y=y)) + geom_path() + theme_minimal()
```

![plot of chunk d2randwalk](/figure/./2016-12-05-random-walks/d2randwalk-1.svg)


