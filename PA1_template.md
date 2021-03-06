# Reproducible Research Course Project 1

This is the Markdown file created for Reproducible Research Course Project 1.  The project involves data from a Fitbit (or equivalent) which records number of steps.  This data set is from an individual user showing the total number of steps taken in each 5 minute interval between October 1st and November 30th 2012.

To work properly, packages are installed in the R Markdown environment.


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(ggplot2)
```

```
## Warning: package 'ggplot2' was built under R version 3.2.4
```

Data is read from the raw csv files provided in the instructions.  The raw file is preserved in its original form while a new data set is created to handle processing.


```r
Data.Raw <- read.csv("activity.csv", header = TRUE)
Data1 <- Data.Raw
names(Data1) <- c("Steps", "Date", "Interval")

Data1$Date <- as.Date(as.character(Data1$Date))
Data1$Time <- as.POSIXct(sprintf("%04d",
    Data1$Interval),format="%H%M")
```

Using processed data, a summary table is made which is then used to produce a histogram showing the total number of steps taken by date during the two month period.


```r
Daily_table <- Data1 %>% group_by(Date) %>% summarise(sum(Steps))
names(Daily_table) <- c("Date", "Steps")

qplot(Steps, data=Daily_table) +
scale_y_continuous(name = "Count", breaks=c(0,2,4,6,8,10)) +
ggtitle("Distribution of daily step totals between 10/1/2012 and 11/30/2012")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

```
## Warning: Removed 8 rows containing non-finite values (stat_bin).
```

![](/unnamed-chunk-3-1.png)<!-- -->

Mean and median steps are calculated with results displayed


```r
mean(Daily_table$Steps, na.rm = TRUE)
```

```
## [1] 10766.19
```

```r
median(Daily_table$Steps, na.rm = TRUE)
```

```
## [1] 10765
```

Another summary table was produced to give the average number of steps over each five minute period throughout the day.  Another histogram is produced to show the result.


```r
Time_table <- Data1 %>% group_by(Interval) %>% summarise(mean(Steps, na.rm=TRUE))
names(Time_table) <- c("Interval", "Steps")

ggplot(Time_table, aes(Interval, Steps)) +
  geom_line() +
  ggtitle("Average number of steps during the day") +
  scale_x_continuous(name = "Time of Day", limits=c(0,2355), breaks=c(0,600,1200,1800,2400))
```

![](/unnamed-chunk-5-1.png)<!-- -->

The 5 minutes interval with the maximum average steps is calculated


```r
filter(Time_table, Steps == max(Steps))[1,1]
```

```
## Source: local data frame [1 x 1]
## 
##   Interval
##      (int)
## 1      835
```

Next we dig deeper into rows where the number of steps data is missing.  This involves the following number of rows.


```r
Data.NA <- filter(Data1, is.na(Steps))
dim(Data.NA)[1]
```

```
## [1] 2304
```

To address missing values, the decision made was to take the median value for each five minute interview throughout the day.  The mean was not chosen since many missing values should be 0, especially in non active parts of the day.  Taking the mode would have resulted in 0 entirely since it would be the most frequent value.  The median provides many 0 values but recognizes steps taken during active parts of the day.

In order to implement this algorithm, the median values by interval are created in a summary table.  That table is then merged back into the data on the interval field.  The final step uses original counts when available and median counts otherwise.  Results are saved in a separate data table from the one with missing values.

Further production is done to calculate the Weekday/Weekend indicator.


```r
Time_table_median <- Data1 %>% group_by(Interval) %>% summarise(median(Steps, na.rm=TRUE))
names(Time_table_median) <- c("Interval", "Steps_Median")

Data2 <- merge(Data1, Time_table_median, by="Interval")
Data2$Steps_Final <- ifelse(is.na(Data2$Steps), Data2$Steps_Median, Data2$Steps)
Data2$Day_of_week <- weekdays(Data2$Date)
Data2$Weekend <- ifelse(Data2$Day_of_week %in% c("Sunday","Saturday"), "Weekend", "Weekday")
```

Similar to above, the average number of steps per day is calculated, this time off the data using median counts instead of NA.  Results are shown in the histogram.


```r
Daily_table2 <- Data2 %>% group_by(Date) %>% summarise(sum(Steps_Final))
names(Daily_table2) <- c("Date", "Steps")

qplot(Steps, data=Daily_table2) +
  scale_y_continuous(name = "Count", breaks=c(0,2,4,6,8,10)) +
  ggtitle("Distribution of daily step totals between 10/1/2012 and 11/30/2012")
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![](/unnamed-chunk-9-1.png)<!-- -->

The revised mean and median is also calculated


```r
mean(Daily_table2$Steps, na.rm = TRUE)
```

```
## [1] 9503.869
```

```r
median(Daily_table2$Steps, na.rm = TRUE)
```

```
## [1] 10395
```

Finally a two level facet grid is produced showing the average steps over each five minute interval split by weekdays vs. weekends.  


```r
Time_table2 <- Data2 %>% group_by(Interval,Weekend) %>% summarise(mean(Steps, na.rm=TRUE))
names(Time_table2) <- c("Interval", "Weekend", "Steps")

ggplot(Time_table2, aes(Interval, Steps)) +
  geom_line() +
  facet_grid(Weekend~.) +
  ggtitle("Average number of steps during the day") +
  scale_x_continuous(name = "Time of Day", limits=c(0,2355), breaks=c(0,600,1200,1800,2400))
```

![](/unnamed-chunk-11-1.png)<!-- -->
