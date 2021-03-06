---
title: "PA1_template.Rmd"
author: "wporter"
date: "June 9, 2014"
output: html_document
---



# Reproducible Research: Peer Assessment 1
## Introduction
It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.



## Data
The data for this assignment can be downloaded from the course web site:

Dataset: Activity monitoring data [52K]
The variables included in this dataset are:

steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

date: The date on which the measurement was taken in YYYY-MM-DD format

interval: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.



## Loading and preprocessing the data
### The following two steps will complete the following
* Load in the packages required in the code to complete the tasks
* Load in the original raw data set
* Change the column formats
* Make an additional data set with the missing values removed

1. Load the data

```r
setwd("~/Documents/Coursera/ReproducableResearch")
require(knitr)
require(plyr)
```

```
## Loading required package: plyr
```

```r
require(lattice)
```

```
## Loading required package: lattice
```

```r
## Save plot in /figure directory
opts_chunk$set(fig.path = "figures/")
## Load the required data
actDf <- read.csv("activity.csv")
```

2. Process/transform the data (if necessary) into a format suitable for your analysis

```r
## Change the columns to the correct formats
actDf$date <- as.Date(actDf$date, format = "%Y-%m-%d")
actDf$steps <- as.numeric(actDf$steps)
actDf$interval <- as.numeric(actDf$interval)
## Make a data frame with NAs removed
tidyDf <- na.omit(actDf)
```

=======




## What is mean total number of steps taken per day?


Make a new data frame with the calculated sum of the number of steps taken each day


```r
## Make a data frame using ddply
dailyDf <- ddply(tidyDf, ~date, summarize, steps = sum(steps, na.rm = T))
```


1. Make a histogram of the total number of steps taken each day


```r
dailyhist <- hist(dailyDf$steps, main = "Total number of steps taken each day with NAs removed", 
    xlab = "Steps/Day", ylim = c(0, 20), breaks = 20)
```

![plot of chunk histogram1](figures/histogram1.png) 


2. Calculate and report the mean and median total number of steps taken per day


```r
mean <- mean(dailyDf$steps, na.rm = T)
median <- median(dailyDf$steps, na.rm = T)
mean
```

```
## [1] 10766
```

```r
median
```

```
## [1] 10765
```


=======



## What is the average daily activity pattern?


Make a new data frame with the calculated mean of the number of steps per interval
accross all days


```r
## Make a data frame with ddply
intDf <- ddply(tidyDf, ~interval, summarize, steps = mean(steps, na.rm = T))
```


1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
## Plot a time series graph
plot(intDf, type = "l", main = "Average number of steps per 5-minute interval", 
    xlab = "5-minute interval", ylab = "Mean number of steps")
```

![plot of chunk lineplot](figures/lineplot.png) 



2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
## Return the interval with the most steps
intDf$interval[which.max(intDf$steps)]
```

```
## [1] 835
```




## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
summary(actDf)
```

```
##      steps            date               interval   
##  Min.   :  0.0   Min.   :2012-10-01   Min.   :   0  
##  1st Qu.:  0.0   1st Qu.:2012-10-16   1st Qu.: 589  
##  Median :  0.0   Median :2012-10-31   Median :1178  
##  Mean   : 37.4   Mean   :2012-10-31   Mean   :1178  
##  3rd Qu.: 12.0   3rd Qu.:2012-11-15   3rd Qu.:1766  
##  Max.   :806.0   Max.   :2012-11-30   Max.   :2355  
##  NA's   :2304
```

```r
sum(is.na(actDf))
```

```
## [1] 2304
```

```r
## All NA values are in the steps column
```


2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

### Strategy
We make a complete data set using the mean steps value for each interval to fill in the missing values in the original data set.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
fillDf <- merge(actDf, intDf, by = "interval")
fillDf$steps.x[is.na(fillDf$steps.x)] <- fillDf$steps.y[is.na(fillDf$steps.x)]
fillDf$steps.x <- round(fillDf$steps.x, digits = 0)
fillDf$steps.y <- NULL
fillDf <- fillDf[, c(2, 3, 1)]
colnames(fillDf) <- c("steps", "date", "interval")
```


4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
dailyfillDf <- ddply(fillDf, ~date, summarize, steps = sum(steps, na.rm = T))
fillhist <- hist(dailyfillDf$steps, main = "Total number of steps taken each day with NAs replaced with means", 
    xlab = "Steps/Day", ylim = c(0, 20), breaks = 20)
```

![plot of chunk histogram2](figures/histogram2.png) 

```r
mean2 <- mean(dailyfillDf$steps, na.rm = T)
median2 <- median(dailyfillDf$steps, na.rm = T)
mean2
```

```
## [1] 10766
```

```r
median2
```

```
## [1] 10762
```

=======


## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?


We use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
fillDf$weekdays <- factor(format(dailyfillDf$date, "%A"))
fillDf$daytype <- fillDf$weekdays
levels(fillDf$daytype) <- list(weekend = c("Saturday", "Sunday"), weekday = c("Monday", 
    "Tuesday", "Wednesday", "Thursday", "Friday"))
```


2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
## Make data frames with avg steps for each interval accross weekends &
## weekdays
lastDf <- ddply(fillDf, interval ~ daytype, transform, avgsteps = mean(steps))
# lastDf <- ddply(fillDf, c('interval', 'daytype'), transform, avgsteps =
# mean(steps))

## Time series plot
xyplot(lastDf$avgsteps ~ lastDf$interval | lastDf$daytype, layout = c(1, 2), 
    type = "l", xlab = "Interval", ylab = "Number of steps", main = "Weekday vs. Weekend Steps")
```

![plot of chunk timeplot](figures/timeplot.png) 



