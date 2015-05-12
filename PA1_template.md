# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
1. The following code loads the data into the **activity** dataframe.

```r
library(dplyr, warn.conflicts=FALSE)
library(ggplot2)
activity <- read.csv("activity.csv")
```
2. No data transformations are needed.

## What is mean total number of steps taken per day?
1. This histogram plots the frequency of total number of steps taken each day.  Per the instructions, missing values in **activity** are ignored.  A binwidth of the range/30 is used.

```r
stepsByDate <- na.omit(activity) %>% group_by(date) %>% summarise(totsteps=sum(steps))
bwidth <- diff(range(stepsByDate$totsteps)) / 30
ggplot(data=stepsByDate, aes(x=totsteps)) +
    geom_histogram(binwidth=bwidth) +
    ggtitle("Frequency of Steps per Day") +
    xlab(paste0("steps/day (binwidth=", bwidth, ")"))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

## What is the average daily activity pattern?



## Imputing missing values
1. The following calculates the total number of rows in **activity** containing NAs ...

```r
sum(is.na(activity$step))
```

```
## [1] 2304
```


## Are there differences in activity patterns between weekdays and weekends?
