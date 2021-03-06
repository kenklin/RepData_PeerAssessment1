# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
1. The following code loads the data into the `activity` dataframe.

```r
library(dplyr, warn.conflicts=FALSE)
library(ggplot2)
library(scales)
activity <- read.csv("activity.csv")
```

2. Change `activity$date` to Date objects.  Create `activity$tod` variable (time on epoch day 1) from `activity$interval` (hhmm) which will be useful for interval displays.

```r
activity$date <- as.Date(activity$date)
activity$tod <- ISOdate(year=1970, month=1, day=1,
    hour=activity$interval %/% 100,
    min=activity$interval %% 100)
```

-------------------------------------------------------------------------------

## What is mean total number of steps taken per day?
1. This histogram plots the frequency of total number of steps taken each day, which is calculated and stored in `stepsPerDay`.  Per the instructions, missing values in `activity` are *ignored*.  Rather than using the default binwidth of `diff(range(stepsPerDay$totsteps))/30` (705.1), a more natural `bwidth` of 500 is used.

```r
# Function that returns a new steps-by-date dataframe from 'activity'
# with a new 'totsteps' variable.  Missing value rows are omitted.
createStepsPerDay <- function(dfActivity) {
  na.omit(dfActivity) %>% group_by(date) %>% summarise(totsteps=sum(steps))
}

# Function that returns a steps-by-date histogram.
plotStepsPerDay <- function(stepsPerDay, title) {
  #bwidth <- diff(range(stepsPerDay$totsteps)) / 30
  bwidth <- 500
  plot <- ggplot(data=stepsPerDay, aes(x=totsteps)) +
    geom_histogram(binwidth=bwidth) +
    ggtitle(title) +
    xlab(paste0("Steps/Day (binwidth=", bwidth, ")")) +
    ylab("Frequency")
  return(plot)
}
```

```r
stepsPerDay <- createStepsPerDay(activity)
plotStepsPerDay(stepsPerDay, "Frequency of Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

2. The mean and median total number of steps taken per day are calculated with this code:

```r
stepsMean <- round(mean(stepsPerDay$totsteps), digits=2)
stepsMedian <- as.integer(median(stepsPerDay$totsteps))
```
The mean is `10766.19` and the median is `10765`.

-------------------------------------------------------------------------------

## What is the average daily activity pattern?
1. This time series plot has the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis).  Missing values in `activity` are ignored.  As a reader aid, interval values have been converted to time of day in `activity$tod`.

```r
# Function that creates steps-by-interval dataset.
createStepsPerInterval <- function(activity) {
  na.omit(activity) %>% group_by(tod, interval) %>% summarise(totsteps=mean(steps))
}

# Function that returns a steps-by-interval time series line plot.  Rather than
# labelling the X axis with a numeric interval, the time of day is used.
plotStepsPerInterval <- function(stepsPerInterval) {
  ggplot(data=stepsPerInterval, aes(x=tod, y=totsteps)) +
    geom_line() +
    ggtitle("Average Steps vs. Time of Day") +
    xlab("Time of Day (5-min intervals)") +
    scale_x_datetime(labels=date_format("%H:%M"),
                     breaks=date_breaks("2 hour")) +
    ylab("Average Steps (across all days)")
}
```

```r
stepsPerInterval <- createStepsPerInterval(activity)
plotStepsPerInterval(stepsPerInterval)
```

![](PA1_template_files/figure-html/unnamed-chunk-7-1.png) 


2. The following code determines which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps.

```r
maxStepsIndex <- which.max(stepsPerInterval$totsteps)
maxSteps <- as.integer(stepsPerInterval[maxStepsIndex, "totsteps"])
maxStepsInterval <- stepsPerInterval[maxStepsIndex, "interval"]
maxStepsTimeOfDay <- format(stepsPerInterval[maxStepsIndex, "tod"], format="%H:%M")
```
The maximum number of steps (206) occurs at 5-minute interval `835` (08:35).

-------------------------------------------------------------------------------

## Imputing missing values
1. The following calculates the total number of rows in `activity` containing NAs ...

```r
numNA <- sum(is.na(activity$steps))
```
The number of NA rows is `2304`.

2. My strategy to fill in all missing (NA) values in the dataset is to use the **mean for that 5-minute interval, previously determined in `stepsPerInterval$totsteps`**.  This was one of the possible strategies suggested in the assignment directions "or the mean for that 5-minute interval, etc."

3. The code below shows how the new `imputed.activity` dataset is created from `activity`, where NA `steps` values are replaced by the mean for that 5-minute interval, which was previously determined in `stepsPerInterval$totsteps`.

```r
# Initialize imputed.activity with activity
imputed.activity <- activity

# Iterate over rows, replacing missing steps with imputed values
for (i in 1:nrow(imputed.activity)) {
  if (is.na(imputed.activity[i, "steps"])) {
    # Determine the interval of the missing steps
    interval <- imputed.activity[i, "interval"]
    
    # Imputed steps are the mean for that 5-min interval.  Since this has already
    # been determined in stepsPerInterval$totsteps, lookup the row by interval.
    imputedSteps <- stepsPerInterval[stepsPerInterval$interval==interval, "totsteps"]
    
    # Overwrite missing steps with the imputed value
    imputed.activity[i, "steps"] <- imputedSteps
  }
}
```

4. A histogram of the total number of steps taken each day is made by calling the previously defined function with `imputed.activity`.

```r
imputed.stepsPerDay <- createStepsPerDay(imputed.activity)
plotStepsPerDay(imputed.stepsPerDay, "Frequency of (Imputed) Steps per Day")
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 

```r
imputedNumNA <- sum(is.na(imputed.activity$steps))
```
The number of NA rows is `0`.

This code again calculates the mean and median total number of steps taken per day.

```r
imputed.stepsMean <- round(mean(imputed.stepsPerDay$totsteps), digits=2)
imputed.stepsMedian <- as.integer(median(imputed.stepsPerDay$totsteps))
```
The mean is `10766.19` and the median is `10766` for the imputed data.

#### Observations
- The two histograms visually appear the same.
- The two *mean* calculations remain the same `10766.19`.  In hindsight, this is to be expected since the chosen strategy of replacing the missing step values with the mean of non-missing values doesn't change the mean!
- The *median* however does shift slightly from `10765` to `10766` due to the increase in observations when imputed data is used.

-------------------------------------------------------------------------------

## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable, `daytype.activity$daytype` with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
weekend <- c("Saturday", "Sunday")
daytype.activity <- activity
for (i in 1:nrow(daytype.activity)) {
  if (weekdays(daytype.activity[i, "date"]) %in% weekend) {
    daytype.activity[i, "daytype"] <- "weekend"
  } else {
    daytype.activity[i, "daytype"] <- "weekday"    
  }
}
```

2. The following panel plot contains a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
daytype.stepsPerInterval <- na.omit(daytype.activity) %>% group_by(tod, daytype) %>% summarise(totsteps=mean(steps))
plotStepsPerInterval(daytype.stepsPerInterval) +
  facet_grid(daytype ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 
