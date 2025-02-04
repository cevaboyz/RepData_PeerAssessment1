---
title: "RepData_PeerAssignment"
author: "Andrea Castagna"
date: "5/14/2021"
output: html_document
keep_md: true
---



## RepData_PeerAssignment on knitr 
======================================

This first chunk of code is simply a function to download the data set for the
assignment, store it in a temporary file, unzip it and read it into the variable 
data, then it proceeds to delete the temporary vector containing the file.



```r
temp <- tempfile()

download.file("https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip", temp)

data <- read.csv(unz(temp, "activity.csv"))

unlink(temp)
```


### What is mean total number of steps taken per day?


The second chunk of code is going to transform and tidy up the data set for the 
next steps:

1. Transform the date column from character class to date class with the package
lubridate and the function ymd(), we also add a column Date in the data frame 
that will be populate by the same dates but transformed with another function:
as.Date;

2. We then proceed to aggregate our steps data with the function aggregate, with 
that we can make a new  data frame  *agg* which is going to have 2 columns: the 
aggregate days and the sum of the steps for each day (NAs were ignored as told);

3. We make an histogram of the total number of steps taken each day;

4. We can then calculate the mean and the median total number of steps taken 
each day. 



```r
require(lubridate)

data$date <- ymd(data$date)

data$Date <- as.Date(data$date)

agg <- aggregate(data$steps, by=list(data$Date), sum)


## Plot

require(ggplot2)

hist(agg$x, 
      xlab="Total number of steps taken each day", 
      ylab="Count", 
      main="Histogram of total number of steps taken each day",
      col=28)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png)

```r
## Mean and Median of the steps

Mean <- mean(agg$x, na.rm = TRUE)

Median <- median(agg$x, na.rm = TRUE)

print(Mean)
```

```
## [1] 10766.19
```

```r
print(Median)
```

```
## [1] 10765
```


### What is the average daily activity pattern?

1. In order to answer this question we need to use the function *aggregate* to 
group each of 5 minute interval across all of the days of detection with the
total mean of steps taken inside the same 5 minute interval.

2. To find the raw with most of the steps we proceed with making a logical 
vector *max_steps_row* which finds the logical position of the row with the max
amount of steps and then it applies it as indexing element for the data set.



```r
step <- aggregate(steps ~ interval, data, mean)

plot(step$interval, step$steps, type = "l", main = "Average steps taken during 5 minute interval", xlab = "Interval", ylab = "Average number of steps")

abline(h = mean(step$steps), col = "red", lwd = 1, lty = 1)

text(13.08599,48.46744, "Mean", col= "red" )
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png)

```r
# Find the row with the most steps 

max_steps_row <- which.max(step$steps)

step[max_steps_row, ]
```

```
##     interval    steps
## 104      835 206.1698
```


### Imputing missing values

This part of the assignment is requiring us to summarize how many NAs are filling the Steps column in our data frame and then proceed to make a new data frame with all of the NAs filled with values that we are going to take from the average number of steps in the same interval from the other day

So, the first step is to check how many NAs we have: ```sum(is.na(data$steps))``` 

The next step is the filling of we are making a copy of our starting data frame: *filleroni* and we then proceed to make a for loop which will check every row of the column *steps* of the *filleroni* data frame if the value is NA, if so it will:

1. take the intervals from the *filleroni$intervals* rows corresponding to the NA in steps and store it inside the *interval_value*, then the vector *steps_value* will be populated from the average steps for the given interval value extracted before;

2. the averaged steps will be used to replace the NAs in the step column from the *filleroni* data frame;

3. We will then use the *aggregate* function on the new data frame to make an aggregated data frame and make an histogram of the *steps*;

4. We will then proceed to computate and confront the mean and median of the first data frame with NAs and the *filleroni* data frame with NAs replaced by the average value of the steps for the given interval

```r
#How many NAs in the data frame

sum(is.na(data$steps))
```

```
## [1] 2304
```

```r
filleroni <- data

for (i in 1:nrow(filleroni)) 
    
    {
        if(is.na(filleroni$steps[i]))
          
        {
            interval_value <- filleroni$interval[i]
            
            steps_value <- step[step$interval == interval_value, ]
            
            filleroni$steps[i] <- steps_value$steps
            
        }
  
    }

filleronidf <- aggregate(steps ~ date, filleroni, sum)


hist(filleronidf$steps, main="Histogram of total number of steps per day with filled NAs", 
     xlab="Total number of steps in a day", col = "green")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png)

```r
Mean_Filled <- mean(filleronidf$steps)

Median_Filled <- median(filleronidf$steps)


Mean <- mean(agg$x, na.rm = TRUE)

Median <- median(agg$x, na.rm = TRUE)



print(Mean_Filled) 
```

```
## [1] 10766.19
```

```r
print(Mean)
```

```
## [1] 10766.19
```

```r
print(Median_Filled) 
```

```
## [1] 10766.19
```

```r
print(Median)
```

```
## [1] 10765
```



### Are there differences in activity patterns between weekdays and weekends?


The last part of the assignment is a comparison among the average steps taken
on weekdays and on weekends, the used data frame was the base data, the one without the filled NAs.



In order to achieve this goal we need to evaluate if a certain day is a weekday e.g. Monday/Tuesday and so on, from Saturday and Sunday. 



We will make a new column *day* which will be populated by the weekdays thanks to the function *weekdays.Date*.



We then proceed to create a two sub data frames *weekdays* and *weekends*, the first one with only days from Monday to Friday, and the other one with only Saturdays and Sundays.



We then use the aggregate function to aggregate steps and intervals from the two data frames and apply the mean function.



With the new *agg1* and *agg22* containing our desired data we can now make a plot for each of those cluster, utilizing the base R plot functions we make the two aforementioned plots with the average steps taken on weekdays and weekends.


```r
data$day <- weekdays.Date(data$date)

suppressWarnings(weekdays <- subset(data, data$day == c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")))

suppressWarnings(weekend <- subset(data, data$day == c("Saturday", "Sunday")))




agg1 <- aggregate(steps ~ interval, weekdays, mean)

agg22 <- aggregate(steps ~ interval, weekend, mean)

par(mfrow=c(2,1))

plot(agg1, type ="l", xlab="Interval", ylab="Number of Steps (Average)" , main="Average steps taken on Weekdays")

plot(agg22, type ="l", xlab="Interval", ylab="Number of Steps (Average)" , main="Average steps taken on Weekends")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png)
