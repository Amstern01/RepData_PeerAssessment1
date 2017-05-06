    knitr::opts_chunk$set(echo = TRUE)

R Markdown
----------

This is an R Markdown document. Markdown is a simple formatting syntax
for authoring HTML, PDF, and MS Word documents. For more details on
using R Markdown see <http://rmarkdown.rstudio.com>.

When you click the **Knit** button a document will be generated that
includes both content as well as the output of any embedded R code
chunks within the document. You can embed an R code chunk like this:

1. Loading and Preprocessing the data
-------------------------------------

    setwd("~/Performance Review/JHU Data Science/Assignments/Reproducible Research")
    fileURL <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(fileURL, destfile = "./datazip.zip")
    unzip("./datazip.zip", exdir = "./datazip")
    data <- read.csv("./datazip/activity.csv", header = TRUE, sep= ",", fill = TRUE)
    data$steps <- as.numeric(data$steps)
    data$date <- as.Date(data$date)
    data$interval <- as.numeric(data$interval)

2. What is mean total number of steps taken per day?
----------------------------------------------------

Calculate total number of steps per day

    StepsPerDay <- aggregate(steps ~ date, data, FUN = sum)

Plot histogram of total number of steps per day

    hist(StepsPerDay$steps)

![](Assignment1_Amstern01_files/figure-markdown_strict/hist-1.png)

Calculate mean and median values of total number of steps per day

    meanStep <- mean(StepsPerDay$steps)
    medianStep <- median(StepsPerDay$steps)
    meanStep <- as.numeric(meanStep)
    medianStep <- as.numeric(medianStep)

The mean steps per day is 1.076618910^{4} and the median steps per day
is 1.076510^{4}.

3. What is the average daily activity pattern?
----------------------------------------------

Calculate the average number of steps per five minute interval

    ActivityPattern <- aggregate(steps ~ interval, data, FUN = mean)

Plot the average number of steps per five minute interval

    plot(ActivityPattern$interval, ActivityPattern$steps, type = "l", main = "Average Number of Steps Per Five Minute Interval", xlab = "interval", ylab= "Average Steps")

![](Assignment1_Amstern01_files/figure-markdown_strict/unnamed-chunk-5-1.png)

Identify 5 minute interval with maximum average number of steps

    library(data.table)
    DT <- as.data.table(ActivityPattern)
    maxInt <- DT[,.SD[which.max(steps)]]
    maxInt <- as.data.table(maxInt)
    maxinterval <- maxInt$interval[1]
    maxsteps <- maxInt$steps[1]

The interval with the highest average number of steps is interval 835 at
206.1698113 steps.

4. Imputing missing values
--------------------------

Calculate number of rows with NAs

    missing <-data[rowSums(is.na(data)) > 0,]
    missingrows <- nrow(missing)

The dataset has 2304 rows with missing values

Imputing missing values based upon the average number of steps in the
interval with the missing value.

    dataImpute <- data
    setDT(dataImpute)[ActivityPattern, stepsi := i.steps, on='interval'][is.na(steps), steps := stepsi][,stepsi:= NULL][]

    ##            steps       date interval
    ##     1: 1.7169811 2012-10-01        0
    ##     2: 0.3396226 2012-10-01        5
    ##     3: 0.1320755 2012-10-01       10
    ##     4: 0.1509434 2012-10-01       15
    ##     5: 0.0754717 2012-10-01       20
    ##    ---                              
    ## 17564: 4.6981132 2012-11-30     2335
    ## 17565: 3.3018868 2012-11-30     2340
    ## 17566: 0.6415094 2012-11-30     2345
    ## 17567: 0.2264151 2012-11-30     2350
    ## 17568: 1.0754717 2012-11-30     2355

Calculate total number of steps per day for imputed dataset

    StepsPerDayImp <- aggregate(steps ~ date, dataImpute, FUN = sum)

Plot histogram of total number of steps per day for imputed dataset

    hist(StepsPerDayImp$steps)

![](Assignment1_Amstern01_files/figure-markdown_strict/unnamed-chunk-10-1.png)

Calculate mean and median values of total number of steps per day for
imputed dataset

    meanImp <- mean(StepsPerDayImp$steps)
    medianImp <- median(StepsPerDayImp$steps)

    DiffMean <- meanStep - meanImp
    DiffMedian <- medianStep - medianImp

The mean steps per day of the imputed data set is 1.076618910^{4} and
the median steps per day of the imputed dataset is 1.076618910^{4}. The
difference in means is 0 and the difference in medians is -1.1886792
between the original and imputed datasets. Imputing the data does not
change the mean number of steps per day but it does slightly alter the
median number of steps per day.

5. Are there differences in activity patterns between weekdays and weekends?
----------------------------------------------------------------------------

Creating factor variable to indicate whether a given date is a weekday
or weekend.

    data[ ,"Weekday"] <-NA
    weekdaynames <- c("Monday", "Tuesday", "Wednesday", "Thursday", "Friday")

    data$Weekday <- ifelse(weekdays(data$date) %in% weekdaynames, 1, 2)

    data$Weekday <- factor(data$Weekday, levels = c(1,2), labels = c("Weekday", "Weekend"))

Plotting average number of steps per five minute interval faceted by
weekday and weekend.

    weekData <- aggregate(steps ~ Weekday+interval, data=data, sum)
    library(ggplot2)

    ## Warning: package 'ggplot2' was built under R version 3.3.2

    wp <- ggplot(weekData, aes(x= interval, y= steps)) + geom_line() 
    wp + facet_grid(Weekday ~ .)

![](Assignment1_Amstern01_files/figure-markdown_strict/unnamed-chunk-13-1.png)
