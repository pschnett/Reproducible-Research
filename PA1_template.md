# Reproducible Research: Peer Assessment 1

#Introduction

It is now possible to collect a large amount of data about personal movement using activity monitoring devices such as a Fitbit, Nike Fuelband, or Jawbone Up. These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

#Data

The data for this assignment can be downloaded from the course web site:

  -Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

  -steps: Number of steps taking in a 5-minute interval (missing values are coded as NA)

  -date: The date on which the measurement was taken in YYYY-MM-DD format

  -interval: Identifier for the 5-minute interval in which measurement was taken


The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

#Assignment

This assignment will be described in multiple parts. You will need to write a report that answers the questions detailed below. Ultimately, you will need to complete the entire assignment in a single R markdown document that can be processed by knitr and be transformed into an HTML file.

Throughout your report make sure you always include the code that you used to generate the output you present. When writing code chunks in the R markdown document, always use `echo = TRUE` so that someone else will be able to read the code. **This assignment will be evaluated via peer assessment so it is essential that your peer evaluators be able to review the code for your analysis.**

For the plotting aspects of this assignment, feel free to use any plotting system in R (i.e., base, lattice, ggplot2)

Fork/clone the [GitHub repository created for this assignment](http://github.com/rdpeng/RepData_PeerAssessment1). You will submit this assignment by pushing your completed files into your forked repository on GitHub. The assignment submission will consist of the URL to your GitHub repository and the SHA-1 commit ID for your repository state.

NOTE: The GitHub repository also contains the dataset for the assignment so you do not have to download the data separately.

**Loading and preprocessing the data**

Show any code that is needed to

1.Load the data (i.e. read.csv())


```r
activityData <- read.csv("activity.csv", colClasses = c("numeric", "character", 
    "numeric"))
summary(activityData)
```

```
##      steps            date              interval     
##  Min.   :  0.00   Length:17568       Min.   :   0.0  
##  1st Qu.:  0.00   Class :character   1st Qu.: 588.8  
##  Median :  0.00   Mode  :character   Median :1177.5  
##  Mean   : 37.38                      Mean   :1177.5  
##  3rd Qu.: 12.00                      3rd Qu.:1766.2  
##  Max.   :806.00                      Max.   :2355.0  
##  NA's   :2304
```
2.Process/transform the data (if necessary) into a format suitable for your analysis

```r
activityData$date <- as.Date(activityData$date, format = "%Y-%m-%d")
activityData$interval <- as.factor(activityData$interval)
```

**What is mean total number of steps taken per day?**

For this part of the assignment, you can ignore the missing values in the dataset.

1.Calculate the total number of steps taken per day


```r
totalStepsPerDay <- aggregate(steps ~ date, data = activityData, sum, na.rm = TRUE)
head(totalStepsPerDay)
```

```
##         date steps
## 1 2012-10-02   126
## 2 2012-10-03 11352
## 3 2012-10-04 12116
## 4 2012-10-05 13294
## 5 2012-10-06 15420
## 6 2012-10-07 11015
```

2.If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.2.2
```

```r
ggplot(totalStepsPerDay, aes(x = steps)) + 
       geom_histogram(fill = "blue", binwidth = 1000) + 
        labs(title="Total steps per day", 
             x = "Number of Steps per Day", y = "Number of intervals in a day") + theme_bw() 
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png) 

3.Calculate and report the mean and median of the total number of steps taken per day

```r
meanStepsPerDay <- mean(totalStepsPerDay$steps)
medianStepsPerDay <-median(totalStepsPerDay$steps)
meanStepsPerDayf <- format(meanStepsPerDay, digits=2, nsmall=2)
medianStepsPerDayf <- format(medianStepsPerDay, digits=2, nsmall=2)
```
The mean is 10766.19 and median is 10765.00.

**What is the average daily activity pattern?**

1.Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
timeSeries <- tapply(activityData$steps, activityData$interval, mean, na.rm = TRUE)
plot(row.names(timeSeries), timeSeries, type = "l", xlab = "5-minute interval", 
    ylab = "Average across all days", main = "Average number of steps taken", 
    col = "blue")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png) 

2.Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
maxIntervals <- which.max(timeSeries)[1]
maxIntervals <- names(maxIntervals)
maxValues <- which.max(timeSeries)[[1]]
```
The 835th 5-minute interval has a maximum of 104 steps.

**Imputing missing values**

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1.Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
noOfNAs <- sum(is.na(activityData))
noOfNAs
```

```
## [1] 2304
```

2.Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
meanOfSteps <- aggregate(steps ~ interval, data = activityData, FUN = mean)
nonNAsteps <- numeric()
for (i in 1:nrow(activityData)) {
    row <- activityData[i, ]
    if (is.na(row$steps)) {
        steps <- subset(meanOfSteps, interval == row$interval)$steps
    } else {
        steps <- row$steps
    }
    nonNAsteps <- c(nonNAsteps, steps)
}
```

3.Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activityData1 <- activityData
activityData1$steps <- nonNAsteps
sum(is.na(activityData1$steps))
```

```
## [1] 0
```

4.Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
stepsPerDay <- aggregate(steps ~ date, activityData1, sum)
colnames(stepsPerDay) <- c("date","steps")
ggplot(stepsPerDay, aes(x = steps)) + 
       geom_histogram(fill = "orange", binwidth = 1000) + 
        labs(title="Steps Taken each Day", 
             x = "Number of Steps per Day", y = "Number of intervals") + theme_bw()
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png) 


```r
stepsMean <- mean(stepsPerDay$steps)
stepsMedian <- median(stepsPerDay$steps)
stepsMeanf <- format(stepsMean, digits=2, nsmall=2)
stepsMedianf <- format(stepsMedian, digits=2, nsmall=2)
```

Before replacing the NAs in the data

-Mean : 10766.19
-Median: 10765.00


After replacing the NAs in the data

-Mean : 10766.19
-Median: 10766.19


**Are there differences in activity patterns between weekdays and weekends?**

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1.Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```r
dayNames <- weekdays(activityData1$date)

dayType <- vector()
for (i in 1:nrow(activityData1)) {
    if (dayNames[i] == "Saturday") {
        dayType[i] <- "Weekend"
    } else if (dayNames[i] == "Sunday") {
        dayType[i] <- "Weekend"
    } else {
        dayType[i] <- "Weekday"
    }
}
activityData1$dayType <- dayType
activityData1$dayType <- factor(activityData1$dayType)

stepsByDayType <- aggregate(steps ~ interval + dayType, data = activityData1, mean)
names(stepsByDayType) <- c("interval", "dayType", "steps")
```

2.Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
library(lattice)

xyplot(steps ~ interval | dayType, stepsByDayType, type = "l", layout = c(1, 2), 
    xlab = "Interval", ylab = "Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png) 
