# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
1. The following code loads the data into the `activity` dataframe.

```r
library(dplyr, warn.conflicts=FALSE)
library(ggplot2)
activity <- read.csv("activity.csv")
```

2. No data transformations are needed.


## What is mean total number of steps taken per day?
1. This histogram plots the frequency of total number of steps taken each day, which is calculated and stored in `stepsByDate`.  Per the instructions, missing values in `activity` are *ignored*.  Rather than using the default binwidth of `diff(range(stepsByDate$totsteps))/30` (705.1), a more natural `bwidth` of 500 is used.

```r
stepsByDate <- na.omit(activity) %>% group_by(date) %>% summarise(totsteps=sum(steps))
#bwidth <- diff(range(stepsByDate$totsteps)) / 30
bwidth <- 500
ggplot(data=stepsByDate, aes(x=totsteps)) +
    geom_histogram(binwidth=bwidth) +
    ggtitle("Frequency of Steps per Day") +
    xlab(paste0("steps/day (binwidth=", bwidth, ")"))
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png) 

2. The mean and median total number of steps taken per day are calculated with this code:

```r
stepsMean <- mean(stepsByDate$totsteps)
stepsMedian <- median(stepsByDate$totsteps)
```
The mean is *1.0766189\times 10^{4}* and the median is *10765*.


## What is the average daily activity pattern?



## Imputing missing values
1. The following calculates the total number of rows in `activity` containing NAs ...

```r
numNA <- sum(is.na(activity$step))
```
The number of NA rows is 2304.

## Are there differences in activity patterns between weekdays and weekends?
