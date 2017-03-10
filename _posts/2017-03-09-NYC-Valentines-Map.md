---
layout: single
title: "Where do New Yorkers go on Valentine's Day?"
category: R
---

If you have a liking for romantic movies, or sitcoms (no shame here!), you would probably have seen at least one or two people found love in New York City. Such as these people who found one another on the Empire State Building: 

![These people who found one another at the Empire State Building](https://typeset-beta.imgix.net/rehost%2F2016%2F9%2F13%2Fa9d29b21-a27e-408a-b335-7997afb9bc93.jpg)

Or this guy who spends 10 years telling his journey to find the woman of his dream:

![](http://static.srcdn.com/wp-content/uploads/2016/10/How-I-Met-Your-Mother-Reunion.jpg)

The interesting question, are New Yorkers really *romantic*, or are they more *romanticized*? Taking advantage of the fact that New Yorkers commute a lot by taxis, and Valentine's Day 2016 was a Sunday, I would analyze the [NYC taxi dataset](http://www.nyc.gov/html/tlc/html/about/trip_record_data.shtml) in order to hopefully uncover their Valentine's Day activities.

To get on the Valentine's Day trip with New Yorkers, first we need to invite all the guests needed for the trip:

```
library(ggplot2)
library(ggmap)
library(plyr)
library(dplyr)
feb = read.csv("valentine_trips")
```




