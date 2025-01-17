---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data


```r
#load dataset from GitHub repo
activity <- read.csv("activity.csv")

#convert date column to Date object
activity$date <- as.Date(activity$date)
```

As the comment code details, the first step is to load the CSV file from the GitHub repo. I did not specify a specific file path as the CSV file is in the same directory as the R Markdown file.

## What is mean total number of steps taken per day?

### Ignoring NAs


```r
#remove all nas from dataset
activity <- activity[complete.cases(activity),]
```
The ```complete.cases``` functions checks if there are any missing values in each row, as the comma suggests. Then it returns a vector of ```TRUE``` or ```FALSE``` values. I then subset the dataframe based on the ```TRUE``` and ```FALSE``` values.


### Calculating Total Steps Per Day


```r
#use tapply to iterate through dataset by using Date column as the index
totalSteps <- tapply(activity$steps, activity$date, sum)
```

I used the ```tapply ``` function to find the ```sum``` of all the steps per day. The resulting vector gives the total amount of steps per day. However, I could have achieved this result by using ```split```. ```tapply```, nevertheless, is more efficient and returns the same result.


### Histogram of Total Steps Per Day


```r
#load the ggplot2 library (as I will be using qplot for the histogram)
library(ggplot2)

qplot(totalSteps, col = "red") + ggtitle("Histogram of Total Steps per Day") + xlab("Steps") + ylab("Frequency of Steps Per Day")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk histTotalSteps](figure/histTotalSteps-1.png)
  
The question asked was somewhat confusing, as it would not be possible to graph both the days and the number of steps in accordance of the days on a histogram. Defined by [Forbes](https://www.forbes.com/sites/naomirobbins/2012/01/04/a-histogram-is-not-a-bar-chart/?sh=168974856d77), "histograms are used to show distributions of variables while bar charts are used to compare variables." Thus, plotting the days in correlation with the number of steps with that day would result in a bar plot. Therefore, I interpreted this question as "show the distribution of frequencies of steps per day". The bar graph lists the ranges of number of steps per day and their frequencies with the days given.

### Calculate Mean and Median of Total Number of Steps per Day


```r
print(mean(totalSteps))
```

```
## [1] 10766.19
```

```r
print(median(totalSteps))
```

```
## [1] 10765
```

Easily, R provides built-in functions ```mean``` and ```median``` to calculate the mean and median of a numeric/integer vector. Simply, I applied this function to the integer vector totalSteps and printed the result onto the console.

## What is the average daily activity pattern?

### Times Series Plot for Intervals


```r
#get all the intervals from dataset
eachInterval <- unique(activity$interval)

#get mean of steps for each interval

meanIntervalSteps <- tapply(activity$steps, activity$interval, mean)

qplot(eachInterval, meanIntervalSteps) + geom_line(aes(col = "red")) + ggtitle("Time Plot of Mean Steps for Each Interval") + xlab("Activity Interval") + ylab("Mean Steps")
```

![plot of chunk timePlotStepsInterval](figure/timePlotStepsInterval-1.png)

The times series plot here plots the mean steps of each interval. The ```tapply``` function takes each interval and then calculates the mean by using the ```mean``` function. Here, I plotted the "mean" points of each interval and then connected them using the built-in ```ggplot2``` library function ```geom_line()```. Then, I customized the labels and the title for the graph to make it more readable for the reader.


### 5-Minute Interval with Maximum Mean Steps


```r
print(names(meanIntervalSteps[meanIntervalSteps == max(meanIntervalSteps)]))
```

```
## [1] "835"
```

## Imputing missing values

### Calculate Total Missing Values


```r
#reread dataset (in the examples above, I got rid of NAs by using complete.cases)

activity <- read.csv("activity.csv")

print(length(activity[is.na(activity)]))
```

```
## [1] 2304
```

Here, I reread the dataset ```activity.csv``` because I had removed missing values from the dataset in the previous examples. Then, I subsetting the missing values from the data activity by using the Boolean Expression ```is.na``` to check for missing values. Then I found the length of the list, which gives the number of NAs in the entire dataset. 

### Strategy for Imputing Missing Values

I plan to assign the missing values with the mean of the day that the value is assigned to. If the mean of the values is not a number (i.e all the numbers belonged to a certain date are ```NA```), then I plan to assign the step value 0.


### Create New Dataset with NAs Filled In


```r
for (row in 1:nrow(activity)) {
   if (is.na(activity[row,]$steps)) {
      meanActivity <- tapply(activity$steps, activity$date, mean, na.rm = TRUE)
      
      
      if (is.nan(meanActivity[[activity[row, ]$date]])) {
         
         activity[row, ]$steps <- 0
      } else {
         
         activity[row, ]$steps <- meanActivity[[activity[row, ]$date]]
      }
      
   }
}

newActivityDf <- activity

print(head(newActivityDf))
```

```
##   steps       date interval
## 1     0 2012-10-01        0
## 2     0 2012-10-01        5
## 3     0 2012-10-01       10
## 4     0 2012-10-01       15
## 5     0 2012-10-01       20
## 6     0 2012-10-01       25
```

```r
#to verify no NAs in dataset
print(length(newActivityDf$steps[is.na(newActivityDf$steps)]))
```

```
## [1] 0
```

I did not loop through the ```date``` and ```interval``` columns of the data set as there were no missing values in either of the datasets. To verify if this is the case, I used the same technique as above to check how many missing values there were in a given column.



```r
#date column
print(length(activity$date[is.na(activity$date)]))
```

```
## [1] 0
```

```r
#interval column
print(length(activity$date[is.na(activity$date)]))
```

```
## [1] 0
```

Both return zero, validating the claim that there are no missing (```NA```) values in each of the dataset columns.

### Updated Histogram of Total Steps Per Day


```r
totalSteps <- tapply(newActivityDf$steps, newActivityDf$date, sum)

#load the ggplot2 library (as I will be using qplot for the histogram)
library(ggplot2)

qplot(totalSteps, col = "red") + ggtitle("Histogram of Total Steps per Day") + xlab("Steps") + ylab("Frequency of Steps Per Day")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk histTotalStepsNATrue](figure/histTotalStepsNATrue-1.png)

There are major differences between the modified (imputed) dataset and the original dataset. In this example, steps with dates that have a mean of ```NaN``` are automatically assigned 0. The estimates in the beginning differ from the ```mean``` and ```median``` that I calculated after imputing the missing values. Therefore, the histogram shows an increase in frequencies of 0. The impact of imputing missing values is that the distribution of frequencies is more "flat", rather than a normal distribution curve.


```r
print(mean(totalSteps))
```

```
## [1] 9354.23
```

```r
print(median(totalSteps))
```

```
## [1] 10395
```

The mean and median of the totalSteps numeric vector decreases, as I have added more 0s to the dataset. The previous numeric vector had lesser amount of elements. With the NAs removed, the mean and median of the data increases due to greater steps (with ```NA``` being gone), and less elements.

## Are there differences in activity patterns between weekdays and weekends?

### Factor Variable for Weekend or Weekday


```r
#convert date column from factor to date to easily identify the weekday/weekend

newActivityDf$date <- as.Date(newActivityDf$date)

weekLab <- weekdays(newActivityDf$date)

weekLab <- sapply(weekLab, function(e) {
   if (grepl("S(at|un)", e)) {
      e <- "weekend"
   } else {
      e <- "weekday"
   }
})

weekLab <- factor(weekLab)

newActivityDf <- cbind(newActivityDf, weekLab)

print(head(newActivityDf))
```

```
##   steps       date interval weekLab
## 1     0 2012-10-01        0 weekday
## 2     0 2012-10-01        5 weekday
## 3     0 2012-10-01       10 weekday
## 4     0 2012-10-01       15 weekday
## 5     0 2012-10-01       20 weekday
## 6     0 2012-10-01       25 weekday
```

### Time Series Plot Split by Weekend and Weekday


```r
week <- data.frame(sapply(split(newActivityDf, newActivityDf$weekLab), function(e) {tapply(e$steps, e$interval, mean, na.rm = TRUE)}))

#complicated version and display is not very user friendly

weekdayDf <- data.frame(tapply(with(newActivityDf, subset(newActivityDf, weekLab == "weekday"))$steps, with(newActivityDf, subset(newActivityDf, weekLab == "weekday"))$interval, mean), weekLab = "weekday")

names(weekdayDf) <- c("Mean Steps", "weekLab")

weekendDf <- data.frame(tapply(with(newActivityDf, subset(newActivityDf, weekLab == "weekend"))$steps, with(newActivityDf, subset(newActivityDf, weekLab == "weekend"))$interval, mean), weekLab = "weekend")

names(weekendDf) <- c("Mean Steps", "weekLab")

weekDf <- rbind(weekdayDf, weekendDf)

qplot(data = weekDf, rownames(weekDf) ,`Mean Steps`) + facet_grid(.~weekLab) + geom_line(aes(group = 1))+ scale_x_discrete(guide = guide_axis(check.overlap = TRUE), labels = rep(unique(activity$interval),2)) + ggtitle("5-Minute Interval Spread Across Weekday and Weekend") + xlab("Interval") + ylab("Number of steps")
```

![plot of chunk weekDayweekEndTimeSeriesInterval](figure/weekDayweekEndTimeSeriesInterval-1.png)

As shown, the plot gives the weekday and weekend steps by interval. I first created two seperate dataframes by the factor variable ```weekLab```. Then I merged the two  after using ```tapply``` from the previous examples. I used a facet grid to show both the weekend and weekday of both intervals. The graph is not very user friendly, as the labels at the bottom are clumped. I used ```scale_x_discrete``` from [this article](https://datavizpyr.com/how-to-dodge-overlapping-text-on-x-axis-labels-in-ggplot2/). However, the labels are again clumped. This was the most feasible option as the labels were overlapping and were illegible. I labeled the graph appropriately with ```ggtitle```, ```xlab```, and ```ylab``` functions. The labels at first were messy and were incorrect, so I overrode the default by using the function ```rep``` and the argument ```unique(activity$interval)```, as it would give a list of all the intervals (no overlaps). I repeated this twice, first for the weekdays plot and then the weekends plot. 
