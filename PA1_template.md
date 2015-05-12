# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
1. The following code loads the data into the **activity** dataframe.

```r
library(dplyr, warn.conflicts=FALSE)
library(ggplot2)
activity <- read.csv("activity.csv")
```
2. Per the assignment instructions, we will ignore NA values in the data set.  I've chosen to ignore all NA rows in all calculations.

```r
activity <- na.omit(activity)
```


## What is mean total number of steps taken per day?
1. This histogram plots the frequency of total number of steps taken each day.

```r
stepsByDate <- activity %>%
    group_by(date) %>%
    summarise(totsteps=sum(steps))
ggplot(data=stepsByDate, aes(x=totsteps)) +
    geom_histogram() +
    ggtitle("Frequency of Steps per Day") +
    xlab("steps / day")
```

```
## stat_bin: binwidth defaulted to range/30. Use 'binwidth = x' to adjust this.
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 

## What is the average daily activity pattern?



## Imputing missing values
1. The following calculates the total number of rows with NAs ...

```r
sum(is.na(activity$step))
```

```
## [1] 0
```


## Are there differences in activity patterns between weekdays and weekends?
