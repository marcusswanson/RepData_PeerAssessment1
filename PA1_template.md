# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data

This section downloads the data from the supplied link [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]
places it in the data folder, and then extracts the activity.csv contained within.  


```r
        library(dplyr)
        library(ggplot2)

        ## Create the data folder
        if(!file.exists("data")) {
                dir.create("data")
        }
        
        ## Download the zip file for the course
        if(!file.exists("./data/project.zip")) {
                print("downloading project.zip...")
                fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
                download.file(fileUrl, destfile = "./data/project.zip", mode = "wb")
                list.files("./data")
                print("downloaded project.zip.")
        }
        
        if(!file.exists("./data/zipcontents")) {
                unzip("./data/project.zip", exdir="./data/zipcontents")
        }
```

We also preprocess the data, and change the date field to be a date column and remove any data where the number of steps is NA


```r
        dataWithNAs <- read.csv("./data/zipcontents/activity.csv", header = TRUE, sep= ",");
        dataWithNAs$date <- as.POSIXct(strptime(dataWithNAs$date, format="%Y-%m-%d"))
        data <- dataWithNAs[!is.na(dataWithNAs$steps),]
```

## What is mean total number of steps taken per day?

### 1. Make a histogram of the total number of steps taken each day

To answer the question, _What is mean total number of steps taken per day?_ can be broken down into a chart showing the total steps per day:


```r
        stepsPerDay <- summarise(group_by(data, date), sum(steps))

        names(stepsPerDay)[1] <- "date"
        names(stepsPerDay)[2] <- "totalSteps"
        stepsPerDayPlot <- ggplot(data=stepsPerDay, aes(x=date, y=totalSteps)) + geom_bar(stat = "identity")
        print (stepsPerDayPlot)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)

### 2. Calculate and report the **mean** and **median** total number of steps taken per day

Which means that the mean total steps per day is calculated by 


```r
        totalStepsPerDayMean <- mean(stepsPerDay$totalSteps)
```

giving a mean value of **10766** and the median is calculated by


```r
        totalStepsPerDayMedian <- median(stepsPerDay$totalSteps)
```

giving a median value of **10765**

## What is the average daily activity pattern?

### 1. Make a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

The following code will create a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
        averageStepsPerInterval <- summarise(group_by(data, interval), mean(steps))

        names(averageStepsPerInterval)[1] <- "interval"
        names(averageStepsPerInterval)[2] <- "averageSteps"

        plot(x=averageStepsPerInterval$interval, xlab = "Interval", y=averageStepsPerInterval$averageSteps, ylab = "Average number of steps taken, averaged across all days", type="l")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)


### 2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

To determine the 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps we can run the following:


```r
       averageStepsPerInterval$interval[which.max(averageStepsPerInterval$averageSteps)]
```

Which gives the interval **835** as shown in the plot above.

## Inputing missing values

Note that there are a number of days/intervals where there are missing
values (coded as `NA`). The presence of missing days may introduce
bias into some calculations or summaries of the data.

### 1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with `NA`s)


```r
        rowsWithNAs <- dataWithNAs[rowSums(is.na(dataWithNAs)) > 0, ]
        nrow(rowsWithNAs)
```

Which gives **2304** rows
        
        
### 2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Only the steps rows have NA values, as shown by


```r
        noOfNAIntervalRows <- nrow(dataWithNAs[is.na(dataWithNAs$interval),])
        noOfNADateRows <- nrow(dataWithNAs[is.na(dataWithNAs$date),])
        noOfNAStepsRows <- nrow(dataWithNAs[is.na(dataWithNAs$steps),])
```

giving the number of NA interval rows as **0**, the number of NA date rows as **0** and the number of NA steps rows as **2304**

### 3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

The new data set can be created by filling in the NA values with the average steps for that interval as so:


```r
        dataWithCompletedNAs <- dataWithNAs
        dataWithCompletedNAs <- merge(dataWithCompletedNAs, averageStepsPerInterval, all.x = TRUE)
        dataWithCompletedNAs$steps <- replace(dataWithCompletedNAs$steps, which(is.na(dataWithCompletedNAs$steps)==TRUE), dataWithCompletedNAs$averageSteps)
```

```
## Warning in replace(dataWithCompletedNAs$steps,
## which(is.na(dataWithCompletedNAs$steps) == : number of items to replace is
## not a multiple of replacement length
```

```r
        dataWithCompletedNAs <- dataWithCompletedNAs[order(dataWithCompletedNAs$date),]
        
        head(dataWithCompletedNAs[1:3])
```

```
##     interval    steps       date
## 1          0 1.716981 2012-10-01
## 63         5 1.716981 2012-10-01
## 128       10 1.716981 2012-10-01
## 205       15 1.716981 2012-10-01
## 264       20 1.716981 2012-10-01
## 327       25 1.716981 2012-10-01
```


### 4. Make a histogram of the total number of steps taken each day and Calculate and report the **mean** and **median** total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

The histogram can be gerenated using the following code:


```r
        stepsPerDayWithCompletedNAs <- summarise(group_by(dataWithCompletedNAs, date), sum(steps))

        names(stepsPerDayWithCompletedNAs)[1] <- "date"
        names(stepsPerDayWithCompletedNAs)[2] <- "totalSteps"
        stepsPerDayWithCompletedNAsPlot <- ggplot(data=stepsPerDayWithCompletedNAs, aes(x=date, y=totalSteps)) + geom_bar(stat = "identity")
```

So with the NA values filled in the histogram looks like
        
![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)

As opposed to the previous version
        
![](PA1_template_files/figure-html/unnamed-chunk-13-1.png)

Which means that the mean total steps per day is calculated by 


```r
        totalStepsPerDayMeanWithCompletedNAs <- mean(stepsPerDayWithCompletedNAs$totalSteps)
```

giving a mean value of **9371.4** as opposed to the previous value of **10766**

and the median is calculated by


```r
        totalStepsPerDayMedianWithCompletedNAs <- median(stepsPerDayWithCompletedNAs$totalSteps)
```

giving a median value of **10395** as opposed to the previous value of **10765**

So yes, adding these values does have an impact of reducing both the mean and the median slightly.

## Are there differences in activity patterns between weekdays and weekends?

For this part the `weekdays()` function may be of some help here. Use
the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
        dataWithCompletedNAs$weekday <- weekdays(dataWithCompletedNAs$date)
        dataWithCompletedNAs$weekday <- factor(dataWithCompletedNAs$weekday, c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"))
        levels(dataWithCompletedNAs$weekday) <- c("weekday", "weekday","weekday","weekday","weekday", "weekend", "weekend")
        head(dataWithCompletedNAs,3)
```

```
##     interval    steps       date averageSteps weekday
## 1          0 1.716981 2012-10-01    1.7169811 weekday
## 63         5 1.716981 2012-10-01    0.3396226 weekday
## 128       10 1.716981 2012-10-01    0.1320755 weekday
```

1. Make a panel plot containing a time series plot (i.e. `type = "l"`) of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis):

The code and plot produced are as follows:


```r
library(lattice)
plotData <- summarise(group_by(dataWithCompletedNAs, weekday, interval), mean(steps))
names(plotData)[3] <- "numberOfSteps"
xyplot(numberOfSteps ~ interval | weekday, type='l',layout=c(1,2), data = plotData, xlab='Interval',ylab='Number of Steps')
```

![](PA1_template_files/figure-html/unnamed-chunk-17-1.png)

So the short answer is **YES**, there are differences between weekday and weekend activity patterns, with weekdays showing more activity in the pre-1000 interval and weekends showing activity more evenly distributed.


