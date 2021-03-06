---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
    
---


## Loading and preprocessing the data


```r
library(dplyr)
library(tibble)
activity <- read.csv(file = "activity.csv", header = TRUE)
activity_df <- tibble::as_tibble(activity) %>%
  mutate(date=as.Date(date))
```

## What is mean total number of steps taken per day?

1. Calculating the total number of steps taken per day

```r
steps_per_day_df <- activity_df %>% 
  group_by(date) %>%
  filter(is.na(steps) == FALSE) %>%
  summarise(No.Of.Steps=sum(steps))
```

2. Histogram of total number of steps taken per day

```r
library(ggplot2)
ggplot(steps_per_day_df, aes(No.Of.Steps)) +
  geom_histogram(bins = 30, na.rm = TRUE)
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

3. Calculating mean and median of the total number of steps taken per day
- The mean is: 

```r
mean(steps_per_day_df$No.Of.Steps)
```

```
## [1] 10766.19
```
- The median is:

```r
median(steps_per_day_df$No.Of.Steps)
```

```
## [1] 10765
```


## What is the average daily activity pattern?
1. A time series plot (i.e. 𝚝𝚢𝚙𝚎 = "𝚕") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)  

```r
dd <- activity_df %>% 
  group_by(interval) %>%
  filter(is.na(steps) == FALSE) %>%
  summarise(avg=mean(steps))
ggplot(dd, aes(x = interval, y = avg)) +
  geom_line()
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
dd %>% 
  top_n(1, avg) %>%
  select(interval)
```

```
## # A tibble: 1 x 1
##   interval
##      <int>
## 1      835
```

## Imputing missing values

1. Calculating the total number of missing values

```r
no_of_missing_values <- activity_df %>% 
  filter(is.na(steps) == TRUE) %>%
  summarise(count=n())
```

The total number of missing values is *2304*

2. Strategy for filling in missing values: Replace missing value with mean value for interval across all days. The function below calculates the average number of steps using the dataset in 1. above

```r
get_interval_mean <- function(int) {
  r <- activity_df %>% 
    filter(is.na(steps) == FALSE, interval == int) %>%
    summarise(avg=mean(steps))
  r$avg
} 
```

3. New dataset with missing values filled in

```r
new_activity_df <- activity_df %>% 
  mutate(steps = ifelse(is.na(steps) == TRUE, 
                        get_interval_mean(interval), steps))
```

4. Histogram of the total number of steps taken each day  

```r
new_steps_per_day_df <- new_activity_df %>% 
  group_by(date) %>%
  filter(is.na(steps) == FALSE) %>%
  summarise(No.Of.Steps=sum(steps))

ggplot(new_steps_per_day_df, aes(No.Of.Steps)) +
  geom_histogram(bins = 30, na.rm = TRUE)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

- The mean is: 

```r
mean(new_steps_per_day_df$No.Of.Steps)
```

```
## [1] 10766.19
```
- The median is:

```r
median(new_steps_per_day_df$No.Of.Steps)
```

```
## [1] 10766.19
```

## Are there differences in activity patterns between weekdays and weekends?

1. Creating a new factor variable indicating whether a given date is a weekday or weekend day.  

```r
df <- new_activity_df %>% 
  mutate(day_type = ifelse(weekdays(date) %in% c("Saturday", "Sunday"), "Weekend", "Weekday"), 
         day_type = factor(day_type, levels=c("Weekend", "Weekday")))
```

2. A plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis)

```r
df2 <- df %>% 
  group_by(day_type, interval) %>%
  summarise(average_steps=mean(steps)) 

ggplot(df2,  aes(x = interval, y = average_steps, group=day_type)) +
  geom_line() +
  facet_grid(day_type ~ .)
```

![](PA1_template_files/figure-html/unnamed-chunk-15-1.png)<!-- -->
