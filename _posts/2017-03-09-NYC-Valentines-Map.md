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

Let's have a peek at the dataset:

```
str(trips)
```
which gives us:
```
'data.frame':	402448 obs. of  14 variables:
 $ pickup_datetime  : Factor w/ 80910 levels "2016-02-14 00:00:00",..: 1 1 1 1 1 1 1 1 2 2 ...
 $ pickup_latitude  : num  40.6 40.8 40.7 40.6 40.7 ...
 $ pickup_longitude : num  -73.8 -74 -74 -73.8 -74 ...
 $ trip_distance    : num  17.14 16.84 1.12 12.55 1.51 ...
 $ passenger_count  : int  1 1 1 1 5 1 2 3 1 1 ...
 $ dropoff_datetime : Factor w/ 82633 levels "2016-02-14 00:00:00",..: 1 1 1 1 391 554 581 753 63 92 ...
 $ dropoff_latitude : num  40.7 40.6 40.8 40.7 40.8 ...
 $ dropoff_longitude: num  -73.8 -73.8 -74 -74 -74 ...
 $ fare_amount      : num  57 52 12 38 9 13 17.5 16.5 5.5 6 ...
 $ tolls_amount     : num  2.08 5.54 0 0 0 0 0 0 0 0 ...
 $ taxes_amount     : num  1.3 0.8 0.8 1.3 1.3 1.3 1.3 1.3 1.3 1.3 ...
 $ tip_amount       : num  0 0 14.08 0 3.09 ...
 $ payment_amount   : num  60.4 58.3 26.9 39.3 13.4 ...
 $ payment_type     : int  2 1 1 2 1 1 1 1 1 1 ...
 ```
 We can notice that the time taken for each trip is missing from the dataset, however we can easily calculate it in R:
 ```
# Convert datetime to POSIXct:
trips$pickup_datetime = as.POSIXct(trips$pickup_datetime)
trips$dropoff_datetime = as.POSIXct(trips$dropoff_datetime)
# Calculate time_taken:
trips$time_taken = as.numeric(trips$dropoff_datetime - trips$pickup_datetime, units = "mins")
```
We also need the rate of fare per mile of each trip:
```
trips$fare_per_mile = trips$fare_amount / trips$trip_distance
```
And of course, the time taken to travel per mile of each trip:
```
trips$time_per_mile = trips$time_taken / trips$trip_distance
```
To increase the effiency of our cleaning process, the next steps would be done by a helper function, data_cleaning.
```
data_cleaning = function(df){
    # filter time outliers:
    df = filter(df, time_taken > 0 & time_taken < 200)
    # filter inaccurate or missing data:
    df = filter(df, trip_distance > 0)
    df = filter(df, passenger_count > 0)
    df = filter(df, pickup_longitude != 0 & pickup_latitude != 0 & dropoff_longitude != 0 & dropoff_latitude != 0)
    # extract the trips that have a reasonable fare per mile*:
    df = filter(df, fare_per_mile >= 2 & fare_per_mile <= 10)
    # extract only credit card (1) and cash (2) payment: 
    df = filter(df, payment_type == 1 | payment_type == 2)
}
```
Looks good! Now we need to apply the function to our whole dataset:
```
trips = data_cleaning(trips)
```
Now let's see how big is the dataset right now:
```
dim(trips)
```
which gives:
```
371619 17
```
More than 30,000 rows (almost 10%) have been gone! Imagine how they would affect our analysis! Now, we would want to calculate the number of dropoffs and pickups for each pair of dropoff coordinates in our dataset:
```
dropoff_counts <- trips %>% 
                    ddply(.(dropoff_longitude, dropoff_latitude), nrow) %>%
                    arrange(desc(V1))   
```
And of course, also for the pickup ones:
```
pickup_counts <- trips %>% 
                    ddply(.(pickup_longitude, pickup_latitude), nrow) %>%
                    arrange(desc(V1))  
```
Checking to see if our chain functions are doing fine:
```
head(dropoff_counts)
```
which returns:
```
dropoff_longitude	dropoff_latitude	V1
-73.95467	        40.82100	        49
-73.75566	        40.71579	        41
-73.93436	        40.75362	        29
-73.92770	        40.77589	        29
-73.91596	        40.83654	        28
-73.97175	        40.74925	        25
```
```
head(pickup_counts)
```
which returns:
```
pickup_longitude	pickup_latitude	V1
-73.95467	        40.82100	      49
-73.75566	        40.71579	      41
-73.93436	        40.75362	      30
-73.92770	        40.77589	      29
-73.91596	        40.83654	      28
-73.97175	        40.74925	      25
```
##The maps of all pickups and dropoffs:

Now is the interesting part: mapping where they go. First, what we need to concern is the [minimum bounding box](https://en.wikipedia.org/wiki/Minimum_bounding_box) of New York City to ensure that the data points that are outside New York City wouldn't be mapped. 

```
min_lat = 40.5774
max_lat = 40.9176
min_long = -74.15
max_long = -73.7004
```
Now we would use some [helper functions](https://github.com/toddwschneider/nyc-taxi-data/tree/master/analysis) from [Todd W. Schneider](http://toddwschneider.com/) to ensure the aesthetic aspect of our maps:
```
alpha_range = c(0.14, 0.75)
size_range = c(0.134, 0.173)
title_with_subtitle = function(title, subtitle = "") {
  ggtitle(bquote(atop(bold(.(title)), atop(.(subtitle)))))
}
```
Wait no more! Here is the map of all the pickup coordinates. Note that the busier the pickup spot is, the lighter it will be on the map. The busiest spots would be in gold color:
```
plot_pickups <- ggplot(aes(x=pickup_longitude,y=pickup_latitude, alpha = V1, size = V1, col = V1),data=pickup_counts) +
            geom_point() +
            scale_x_continuous(limits=c(min_long, max_long)) +
            scale_y_continuous(limits=c(min_lat, max_lat)) +
            theme(panel.background = element_rect(fill = 'black', colour = 'black'), 
                  panel.grid.major = element_blank(), 
                  panel.grid.minor = element_blank())+
            scale_color_gradient(low="white", high="#ffd700", trans = "log") +
            scale_alpha_continuous(range = alpha_range, trans = "log", limits = range(pickup_counts$V1)) +
            scale_size_continuous(range = size_range, trans = "log", limits = range(pickup_counts$V1)) +
            theme(axis.text.x=element_blank()) +
            theme(axis.text.y=element_blank()) +
            theme(axis.title.y=element_blank()) +
            theme(axis.title.x=element_blank()) +
            theme(legend.position="none") +
            title_with_subtitle("New York City Taxi Pickups", "Valentine's Day 2016") +
            theme(plot.title=element_text(hjust = 0.5, margin=margin(b = -100, unit = "pt"), colour = "white"))
 ```
 
