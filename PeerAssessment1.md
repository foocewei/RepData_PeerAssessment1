---
title: "Reproducible Research - Peer Assessment 1"
author: "foocewei"
date: "Sunnday, December 14, 2014"
output: html_document
---
```
require("knitr")
opts_chunk$set(cache=TRUE, cache.path = 'test_files/', fig.path='figure/')
```
# Reproducible Research - Peer Assessment 1


# Loading and preprocessing the data
## Show any code that is needed to
## 1. Load the data (i.e. `read.csv()`)
### Assumption: source file is already unzipped into working directory

```r
data <- read.csv("activity.csv")
```

## 2. Process/transform the data (if necessary) into a format suitable 
## for your analysis
### aggregate data by date

```r
data$date <- as.Date(data$date)
```

# What is mean total number of steps taken per day?
## For this part of the assignment, you can ignore the missing values in
## the dataset.
## 1. Make a histogram of the total number of steps taken each day

```r
steps <- aggregate(x=data$steps, by=list(data$date), FUN=sum, na.rm=TRUE)
names(steps) <- c("date", "steps")

hist(steps$steps,
     main="Total number of steps taken per day",
     xlab="Steps per day", 
     ylab="Frequency",
     breaks=20)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

## 2. Calculate and report the **mean** and **median** total number of steps 
## taken per day
### Mean total number of steps taken per day

```r
mean(steps$steps, na.rm=TRUE)
```

```
## [1] 9354.23
```

### Median total number of steps taken per day

```r
median(steps$steps, na.rm=TRUE)
```

```
## [1] 10395
```

# What is the average daily activity pattern?
## 1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval 
## (x-axis) and the average number of steps taken, averaged across all days 
## (y-axis)

```r
interval <- aggregate(x=data$steps, by=list(data$interval), 
                      FUN=mean, na.rm=TRUE)
names(interval) <- c("interval", "steps")

plot(x=data$interval,
     y=data$steps,
     type="l",
     main="Average number of steps taken per 5 minute interval",
     xlab="Interval",
     ylab="Number of Steps")
```

![plot of chunk unnamed-chunk-6](figure/unnamed-chunk-6-1.png) 

## 2. Which 5-minute interval, on average across all the days in the dataset, 
## contains the maximum number of steps?

```r
interval[which.max(interval$steps), "interval"]
```

```
## [1] 835
```

# Imputing missing values
## Note that there are a number of days/intervals where there are missing
## values (coded as `NA`). The presence of missing days may introduce
## bias into some calculations or summaries of the data.

## 1. Calculate and report the total number of missing values in the dataset 
## (i.e. the total number of rows with `NA`s)

```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

## 2. Devise a strategy for filling in all of the missing values in the dataset. 
## The strategy does not need to be sophisticated. For example, you could use 
## the mean/median for that day, or the mean for that 5-minute interval, etc.
### Missing values are replaced with the mean for that interval

## 3. Create a new dataset that is equal to the original dataset but with the 
## missing data filled in.

```r
library(plyr)

impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))

data.imputed <- ddply(data, ~interval, transform, steps = impute.mean(steps))
```

## 4. Make a histogram of the total number of steps taken each day and 
## Calculate and report the **mean** and **median** total number of steps 
## taken per day. Do these values differ from the estimates from the first 
## part of the assignment? What is the impact of imputing missing data on the 
## estimates of the total daily number of steps?

```r
data.imputed.date <- aggregate(x=data$steps, by=list(data$date), 
                               FUN=sum, na.rm=TRUE)
names(data.imputed.date) <- c("date", "steps")

hist(data.imputed.date$steps,
     breaks=20,
     main="Total number of steps per day",
     xlab="Steps per day",
     ylab="Frequency")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

# Are there differences in activity patterns between weekdays and weekends?
## For this part the `weekdays()` function may be of some help here. Use
## the dataset with the filled-in missing values for this part.

## 1. Create a new factor variable in the dataset with two levels -- "weekday" 
## and "weekend" indicating whether a given date is a weekday or weekend day.
### Create new variable to store whether it is a weekday or weekend

```r
data.imputed$day <- as.factor(ifelse(weekdays(data.imputed$date) %in% 
                                c("Saturday","Sunday"), "Weekend", "Weekday"))
```

## 2. Make a panel plot containing a time series plot (i.e. `type = "l"`) 
## of the 5-minute interval (x-axis) and the average number of steps taken, 
## averaged across all weekday days or weekend days (y-axis). The plot should 
## look something like the following, which was created using **simulated 
## data**:
##        ![Sample panel plot](instructions_fig/sample_panelplot.png)

```r
library(ggplot2)

data.imputed.interval <- aggregate(x=data.imputed$steps, 
                                   by=list(data.imputed$interval, data.imputed$day),
                                   FUN=mean, na.rm=TRUE)
names(data.imputed.interval) <- c("interval", "day", "steps")

plot <- ggplot(data=data.imputed.interval, aes(x=interval, y=steps)) 
plot <- plot + 
        ggtitle("Time series plot of average steps by interval") +
        geom_line(size=1) +
        facet_wrap(~day,nrow=2)
plot
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 
