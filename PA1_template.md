---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---



## Overview

This program analyzes a dataset as part of the Johns Hopkins Coursera Reproducible Research course fulfilling Course Project 1.

The raw data, provided in a zipped csv format, and all materials for this assignment are available for review in my github repository <https://github.com/drose99/RepData_PeerAssessment1>.

The analysis and this Rmd document contain all of the code and output for the eight requested analyses.  Note:  the code for generating the requested plots has been suppressed in the html output, but may be viewed in the source Rmd file.  The eight code blocks are are:

1. Code for reading in the dataset and/or processing the data
2. Histogram of the total number of steps taken each day
3. Mean and median number of steps taken each day
4. Time series plot of the average number of steps taken
5. The 5-minute interval that, on average, contains the maximum number of steps
6. Code to describe and show a strategy for imputing missing data
7. Histogram of the total number of steps taken each day after missing values are imputed
8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends

### Read and Pre-process data

This code reads the data from the github respository, unzips and saves it to an object 'raw'.  Please note, users of this code will need to modify the path to where they would like the zipped files saved to their computer.


```r
library(data.table)
URL <- "https://github.com/drose99/RepData_PeerAssessment1/blob/master/activity.zip"
path <- "c:/users/doug rose/documents/"
zip_filename <- "activity.zip"
unzip_filename <- "activity.csv"
download.file(URL, paste(path,zip_filename,sep=""))
raw <- read.csv("c:/users/doug rose/documents/activity.csv")   
```

## Histogram of steps by day

Histogram of the count of days where the total number of steps falls into one of five buckets.   The code aggregates steps to the day level before plotting the histogram.  The histogram uses the default breaks for the number of steps in each bucket.

```r
steps_by_day <- aggregate(raw$steps,list(raw$date),sum)
hist(steps_by_day$x, xlab = "Steps by Day", main = "Histogram of Steps by Day")
```

![](PA1_template_files/figure-html/steps-1.png)<!-- -->

Analysis:  The number of steps by day is roughly normal with a range from approximately 2,500 to 22,500.  The median for this 2-month sample occuring approximately 27 times were step counts between 10,000 and 15,000.

## Mean and median number of steps by day

Use the base r 'summary' function to provide the summary statistics including mean and median.

```r
summary(as.numeric(steps_by_day$x))
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
##      41    8841   10765   10766   13294   21194       8
```

The quantitative elements of the analysis above are roughly correct based on the summary statistics.

## Time series plot of the average number of steps taken

Calculate the mean number of steps for each interval excluding NA values in a new data frame (avg_stps_by_interval.  Plot the interval vs. mean number of steps as a trend line.

```r
library(ggplot2)
avg_stps_by_interval <- as.data.frame(cbind(
        unique(raw$interval),
        tapply(raw$steps, raw$interval, mean, na.rm = TRUE)))
colnames(avg_stps_by_interval) <- c("interval","mean_steps")
ggplot(avg_stps_by_interval,aes(interval,mean_steps)) +
    geom_line() +
    labs(title = "Steps by 5 Minute Interval", x= "Interval (hhmm)", y="Mean Steps") +
    theme(plot.title = element_text(hjust = 0.5), plot.margin=unit(c(1,1,1.5,1.2),"cm")) 
```

![](PA1_template_files/figure-html/time_series-1.png)<!-- -->

Analysis:  The number of steps is a multi-modal distribution with clear peaks (>90 steps in 5 minutes) around 08:00, 12:00, 16:00 and 19:00.  The 0800 peak is particularly pronounced and broad.  Very few steps are taken between 2200 and 0500 as would be expected during typical sleeping hours.

## The 5-minute interval that, on average, contains the maximum number of steps

Use the default r max function on the mean_steps column of the avg_stps_by_interval data frame.

```r
subset(avg_stps_by_interval,
       avg_stps_by_interval$mean_steps==max(avg_stps_by_interval$mean_steps,na.rm=TRUE))
```

```
##     interval mean_steps
## 835      835   206.1698
```

Analysis:  As observed in the graph, the peak of the 0800 peak is during the 8:35 interval at 206 steps per five minute interval on average.

## Code to describe and show a strategy for imputing missing data

Apply a simple strategy assuming no daily variation and hourly patterns are consistent each day.  

Create a new imputed data frame (imp) from the raw data (raw) with a variable (imp_steps) containing either the corresponding mean value from that 5-minute interval (avg_stps_by_interval) if the raw value is NA or actual value.

```r
imp <- merge(raw,avg_stps_by_interval, by = "interval")
for(i in 1:nrow(imp)){
    if(is.na(imp$steps[i])) {
    imp$imp_steps[i] <- imp$mean_steps[i] 
    } else { imp$imp_steps[i] <- imp$steps[i]
    }
}
head(imp)
```

```
##   interval steps       date mean_steps imp_steps
## 1        0    NA 2012-10-01   1.716981  1.716981
## 2        0     0 2012-11-23   1.716981  0.000000
## 3        0     0 2012-10-28   1.716981  0.000000
## 4        0     0 2012-11-06   1.716981  0.000000
## 5        0     0 2012-11-24   1.716981  0.000000
## 6        0     0 2012-11-15   1.716981  0.000000
```

Analysis:  This simplistic imputation heuristic may understate some days or overstate others to the extent there is day-of-week patterns in step counts.  A better, more complicated analysis as potential for future work might be to take the mean of the two intervals before and after the missing data.  If multiple sequential data points are missing, a regression line between the next available points might be appropriate.

## Histogram of the total number of steps taken each day after missing values are imputed

Histogram of the count of days where the total number of steps (actual or imputed) falls into one of five buckets.   The code aggregates steps to the day level before plotting the histogram.  The histogram uses the default breaks for the number of steps in each bucket.

```r
imp_steps_by_day <- aggregate(imp$imp_steps,list(imp$date),sum)
hist(imp_steps_by_day$x, xlab = "Steps by Day", main = "Histogram of Steps by Day (NAs Imputed)")
```

![](PA1_template_files/figure-html/hist2-1.png)<!-- -->

Analysis: comparison of the histogram excluding incomplete data and using imputed data only shows a material difference in the number of days observing steps between 10,000 and 15,000 increased while the number of days for all of the other buckets stayed the same.  This is intuitive.  In the dataset there are eight days with no observations.  It makes sense that the imputed distribution of steps in these days would follow the distribution of the rest of the sample with the majority of the days falling in the median bucket.

## Panel plot comparing average steps by interval for weekdays and weekends

Using average steps by interval data (NAs imputed), segregate weekday from weekend observations.  Append new dow, weekend and weekday indicator variables to raw.  Aggregate steps to the interval and weekend/weekday indicator level and plot weekend/weekday on separate charts.

```r
imp$dow <- strftime(imp$date, format = "%u")
for(i in 1:nrow(imp)) {
     if(imp$dow[i] == 6 || imp$dow[i] == 7) {
     imp$wdwe[i] <- "weekend"
          } else { imp$wdwe[i] <- "weekday"
        }
}
avg_stps_by_interval_dow <- aggregate(imp$imp_steps, list(imp$interval, imp$wdwe), mean)
colnames(avg_stps_by_interval_dow) <- c("interval","wdwe","mean_steps")
ggplot(avg_stps_by_interval_dow, aes(interval,mean_steps)) + 
    geom_line() + 
    facet_grid(. ~ wdwe) +
    labs(title = "Mean Steps by Type of Day/Interval\n1Oct2012 - 30Nov2012", x = "Interval (hhmm)") +
    theme(plot.title = element_text(hjust = 0.5), plot.margin=unit(c(1,1,1.5,1.2),"cm")) 
```

![](PA1_template_files/figure-html/dow-1.png)<!-- -->

Analysis:  There are fewer and lower distinct peaks in the weekend data relative to the weekday data as would be expected by weekend activities which are less driven by a work schedule.  In addition, weekend step uptick later in the day than on weekdays.
