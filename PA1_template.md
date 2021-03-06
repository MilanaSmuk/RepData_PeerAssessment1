# Reproducible Research: Peer Assessment 1
Milana Smuk  
11 May 2017  

## R Markdown Assignment 1

This is a R Markdown document writen as a part of Assignment Project 1 for Reproducible Research Course by [Johns Hopkins Bloomberg School of Public Health](http://www.jhsph.edu/) via [Coursera](https://coursera.org).
  
  This Assignment contains 5 parts showing 9 steps of full submission:


1. Loading and preprocessing the data 
2. Histogram of the total number of steps taken each day 
3. Mean and median number of steps taken each day
4. Time series plot of the average number of steps taken
5. The 5-minute interval that, on average, contains the maximum number of steps
6. Code to describe and show a strategy for imputing missing data
7. Histogram of the total number of steps taken each day after missing values are imputed
8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

Every part contains the R code needed to reproduce the results (numbers, plots, etc.) in the report

## Loading and preprocessing the data

### 1. Loading and preprocessing the data 

 - Read in the data set:

```r
if(!file.exists("activity.csv")){
    unzip("activity.zip")
}
data <- read.csv("activity.csv", header = TRUE, sep = ',')
```

 - formating the date variable as a POSIXct type: 


```r
data$date <- as.POSIXct(data$date, format="%Y-%m-%d")
```


## What is mean total number of steps taken per day?

### 2. Histogram of the total number of steps taken each day

First we calculate total number of steps for each day, and then make a histogram 


```r
steps.total <- aggregate(data$steps, by = list(data$date), sum, na.rm = TRUE)
names(steps.total) <- c("date", "total")
```


```r
hist(steps.total$total, main = "Total number of steps taken each day", xlab = "Steps per day", col= "lightgreen", breaks = 15, ylim= c(0,20))
```

![](PA1_template_files/figure-html/histogram1-1.png)<!-- -->


### 3. Mean and median number of steps taken each day


```r
mean.steps <- mean(steps.total$total, na.rm = TRUE)
median.steps <- median(steps.total$total, na.rm = TRUE)
```

Mean of total steps:

```r
mean.steps
```

```
## [1] 9354.23
```

Median of total steps:

```r
median.steps
```

```
## [1] 10395
```


## What is the average daily activity pattern?

### 4. Time series plot of the average number of steps taken

First we make a table with calculated values of average steps taken in every 5-minute interval, and then make a time series plot


```r
pattern <- aggregate( x = list(steps = data$steps), by = list(interval = data$interval),  mean, na.rm=TRUE)
plot(x=pattern$interval, y=pattern$steps, type="l", xlab="5-minute interval", ylab = "Average number of steps", main = "Average number of steps per 5-minute interval", lwd=2)
```

![](PA1_template_files/figure-html/time_series_plot-1.png)<!-- -->


### 5. The 5-minute interval that, on average, contains the maximum number of steps


```r
pattern[which.max(pattern$steps), ]
```

```
##     interval    steps
## 104      835 206.1698
```

## Imputing missing values

### 6. Code to describe and show a strategy for imputing missing data

Quick check how many missing values are there in steps-column:


```r
sum(is.na(data$steps))
```

```
## [1] 2304
```

The idea is to make a new data set and replace missing values in steps - column with an average value of 5-minute interval where missing value occurs.  


```r
data_new <- data
nas <- is.na(data_new$steps)
avg <- tapply(data_new$steps, data_new$interval, mean, na.rm= TRUE, simplify = TRUE)
data_new$steps[nas] <- avg[as.character(data_new$interval[nas])]
```

Quick check of  missing values after data transformation:


```r
sum(is.na(data_new$steps))
```

```
## [1] 0
```



### 7. Histogram of the total number of steps taken each day after missing values are imputed

We are repeating the process described above under Nr.2, with the difference of using newly created data set with replaced missing values in steps - column:


```r
steps.total2 <- aggregate(data_new$steps, by = list(data_new$date), sum)
names(steps.total2) <- c("date", "total")
hist(steps.total2$total, main = "Total number of steps taken each day", xlab = "Steps per day", col= "lightgreen", breaks = 20, ylim= c(0,20))
```

![](PA1_template_files/figure-html/histogram_after-1.png)<!-- -->


Mean of the steps taken:

```r
mean(steps.total2$total)
```

```
## [1] 10766.19
```

Median of the steps taken:


```r
median(steps.total2$total)
```

```
## [1] 10766.19
```

The results are different from those given in Subchapter 2. 

## Are there differences in activity patterns between weekdays and weekends?

### 8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

- Add a new column in the data set with replaced missing values. That is a factor variable containing two values - weekday (for days from Monday to Friday) and weekend (for Saturday and Sunday)


```r
library(dplyr)
data_new <- mutate(data_new, daytype = ifelse(weekdays(data_new$date) == "Saturday" | weekdays(data_new$date) == "Sunday", "weekend", "weekday"))
data_new$daytype <- as.factor(data_new$daytype)
```

- Makes time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis).


```r
library(ggplot2)
avg2 <- aggregate(steps ~ interval + daytype, data = data_new, mean)
ggplot(avg2, aes(interval, steps, color = daytype)) + geom_line() + facet_grid(daytype ~ .) + 
        xlab("5-minute interval") + ylab("Number of steps")
```

![](PA1_template_files/figure-html/time_series_plot2-1.png)<!-- -->

