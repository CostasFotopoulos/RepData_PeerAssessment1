---
title: "Reproducible Research Course Project 1"
author: "Costas Fotopoulos"
date: "July 21, 2019"
output: 
      html_document:
            keep_md: TRUE
---



## Introduction


It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the “quantified self” movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.


This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.


The data for this assignment can be downloaded from the course web site:

* Dataset: [Activity monitoring data [52K]](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip)  

The variables included in this dataset are:

* **steps**: Number of steps taking in a 5-minute interval (missing values are coded as $\color{red}{\text{NA}}$)
* **date**: The date on which the measurement was taken in YYYY-MM-DD format
* **interval**: Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## 1. Code for reading in the dataset and/or processing the data

### Reading dataset

1. Download the data

```r
if(!file.exists('activity.zip')){
      download.file('https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip', 
                    destfile = 'activity.zip')
}
```
2. Unzip data

```r
if (!file.exists('activity.csv')){
      unzip('activity.zip')
}
```
3. Assign data to variable ```activity``` and preview them with ```str()``` function

```r
activity <- read.csv('activity.csv')
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

### Processing data

1. Fix type of data in date column

```r
activity$date <- as.Date(activity$date, format = "%Y-%m-%d")
str(activity)
```

```
## 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
```

## 2. What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day and remove NA values.

```r
totalStepsDay <- with(activity, aggregate(steps, by = list(date), FUN = sum, na.rm = TRUE))
names(totalStepsDay) <- c('date', 'total.steps')
```
2. Make a histogram of the total number of steps taken each day

```r
hist(totalStepsDay$total.steps, xlab = 'Total steps',
     main = 'Total steps per day', col = 'red', ylim = c(0,30),
     breaks = seq(0,28000,by=2355))
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

3. Calculate and report the mean and median of the total number of steps taken per day
* Mean

```r
mean(totalStepsDay$total.steps)
```

```
## [1] 9354.23
```

* Median

```r
median(totalStepsDay$total.steps)
```

```
## [1] 10395
```

## 3. What is the average daily activity pattern?

1. Make a time series plot (i.e. $\color{red}{\text{type="l"}}$) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

Calculate average daily activity pattern and remove NAs:

```r
averageActivityDay <- aggregate(activity$steps, by = list(activity$interval), FUN = mean, na.rm = TRUE)
names(averageActivityDay) <- c('interval', 'mean')
```

Plot average daily activity:

```r
plot(averageActivityDay$interval, averageActivityDay$mean, type = 'l', col = 'blue', 
     main = 'Average number of steps per intervals',
     xlab="Interval",
     ylab="Average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
averageActivityDay[which.max(averageActivityDay$mean), ]$interval
```

```
## [1] 835
```

## 4. Imputing missing values
There are a number of days/intervals where there are missing values (coded as $\color{red}{\text{NA}}$). The presence of missing days may introduce bias into some calculations or summaries of the data.

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with $\color{red}{\text{NA}}$s)

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```
Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
imputedSteps <- averageActivityDay$mean[match(activity$interval, averageActivityDay$interval)]
```
Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activityImuted <- transform(activity, steps = ifelse(is.na(activity$steps),
                                                     yes = imputedSteps,
                                                     no = activity$steps))
totalStepsImuted <- aggregate(steps ~ date, activityImuted, sum)
names(totalStepsImuted) <- c('date', 'daily.steps')
```
Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
hist(totalStepsImuted$daily.steps, xlab = 'Total steps',
     main = 'Total steps per day',
     col = 'red',
     breaks = seq(0,28000,by=2355),
     ylim = c(0,30))
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->

* mean

```r
mean(totalStepsImuted$daily.steps)
```

```
## [1] 10766.19
```
* median

```r
median(totalStepsImuted$daily.steps)
```

```
## [1] 10766.19
```

## 5. Are there differences in activity patterns between weekdays and weekends?
For this part the $\color{red}{\text{weekdays()}}$ function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
activity$weekday <- sapply(activity$date, function(x){
      if(weekdays(x)=='Saturday' | weekdays(x) == 'Sunday'){
            weekdayType <- 'weekend'
      }
      else {
            weekdayType <- 'weekday'
      }
      weekdayType
})
str(activity)
```

```
## 'data.frame':	17568 obs. of  4 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  $ weekday : chr  "weekday" "weekday" "weekday" "weekday" ...
```

2. Make a panel plot containing a time series plot (i.e. $\color{red}{\text{type = "l"}}$) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).

```r
activityByDate <- aggregate(steps~interval + weekday, activity, mean, na.rm = TRUE)
library(ggplot2)
plot <- ggplot(activityByDate, aes(x=interval, y = steps, color = weekday)) +
      geom_line() +
      labs(title = 'Average daily steps for weekdays and weekends',
           x='Interval',
           y='Average number of steps') +
      theme(plot.title = element_text(hjust = 0.5)) +
      facet_wrap(~weekday, ncol = 1, nrow = 2)
print(plot)
```

![](PA1_template_files/figure-html/unnamed-chunk-19-1.png)<!-- -->
