---
layout: post
title:  "Exploratory Data Analysis Of Bike Sharing Demand"
date:   2018-02-24 22:40:47 +0800
categories: jekyll update
---

Introduction
------------

**Bike Sharing Demand** 是 **Kaggle** 上的一個已結束的比賽題目，比賽目的是**預測**城市裡共享單車系統**逐時**的**被租賃數目**。

自從幾年前遇到這個資料以後，它就被我拿來當作反覆練習的材料，因為我認為這個資料集有以下優點：

-   它是真實資料，跟一般教科書上用來練習演算法的完美資料不同，我們必須親手整理它，所以在進行特徵工程的時候，可以用各種不同的角度來建構這個資料。
-   雖帶有時間序列的結構，但**逐時**之間又可以看到清楚的模式，所以我們可以同時練習不同類型的演算法來預測。

以此文紀錄我如何實做 **Bike Sharing Demand** 這個資料的探索性分析

內容大綱
--------

-   資料集介紹
-   使用套件
-   載入資料
-   資料維度及內容
-   特徵工程: 新增時間特徵
-   為特徵設定合理的變數型態
-   觀察反應變數
-   觀察類別變數與反應變數的關係
-   觀察連續變數與反應變數的關係
-   其他角度的觀察
-   小結

資料集介紹
----------

[Bike Sharing Demand From Kaggle](https://www.kaggle.com/c/bike-sharing-demand/data)

You are provided hourly rental data spanning two years. For this competition, the training set is comprised of the first 19 days of each month, while the test set is the 20th to the end of the month. You must predict the total count of bikes rented during each hour covered by the test set, using only information available prior to the rental period.

**Data Fields**

-   `datetime`: hourly date + timestamp (時間標注)
-   `season`: (季節)
    -   1 = spring (春)
    -   2 = summer (夏)
    -   3 = fall (秋)
    -   4 = winter (冬)
-   `holiday`: whether the day is considered a holiday (是否為假日)
-   `workingday`: whether the day is neither a weekend nor holiday (是否為工作日)
-   `weather`: (天氣)
    -   1: Clear, Few clouds, Partly cloudy, Partly cloudy （晴朗少雲）
    -   2: Mist + Cloudy, Mist + Broken clouds, Mist + Few clouds, Mist (多雲)
    -   3: Light Snow, Light Rain + Thunderstorm + Scattered clouds, Light Rain + Scattered clouds (雷雨)
    -   4: Heavy Rain + Ice Pallets + Thunderstorm + Mist, Snow + Fog (極端天氣)
-   `temp`: temperature in Celsius (溫度)
-   `atemp`: "feels like" temperature in Celsius (體感溫度)
-   `humidity`: relative humidity (濕度)
-   `windspeed`: wind speed (風速)
-   `casual`: number of non-registered user rentals initiated (未註冊用戶租賃數量)
-   `registered`: number of registered user rentals initiated (註冊用戶租賃數量)
-   `count`: number of total rentals (總租賃數量)

使用套件
--------

``` r
library(magrittr)
library(dplyr)
library(tidyr)
library(lubridate)
library(ggplot2)
library(GGally)
```

載入資料
--------

``` r
train <- read.csv(file = "./data/train.csv", stringsAsFactors = FALSE)
```

資料維度及內容
--------------

-   可以看到初始載入的資料一共有 `10886`筆資料，且有 `12` 個變數。
-   有些特徵的變數型態必須要調整，如 `season`、`holiday`、`workingday`、`weather`等，必須是類別型(*factor*)而非整數型(*integer*)

``` r
str(train)
```

    ## 'data.frame':    10886 obs. of  12 variables:
    ##  $ datetime  : chr  "2011-01-01 00:00:00" "2011-01-01 01:00:00" "2011-01-01 02:00:00" "2011-01-01 03:00:00" ...
    ##  $ season    : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ holiday   : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ workingday: int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ weather   : int  1 1 1 1 1 2 1 1 1 1 ...
    ##  $ temp      : num  9.84 9.02 9.02 9.84 9.84 ...
    ##  $ atemp     : num  14.4 13.6 13.6 14.4 14.4 ...
    ##  $ humidity  : int  81 80 80 75 75 75 80 86 75 76 ...
    ##  $ windspeed : num  0 0 0 0 0 ...
    ##  $ casual    : int  3 8 5 3 0 0 2 1 1 8 ...
    ##  $ registered: int  13 32 27 10 1 1 0 2 7 6 ...
    ##  $ count     : int  16 40 32 13 1 1 2 3 8 14 ...

特徵工程: 新增時間特徵
----------------------

此資料為時間序列型結構的資料，但`datetime`給我們的是像 `"2011-01-01 00:00:00"`這樣的紀錄，為了分析的方便，我們必須拆解他，將日期與時間獨立取出，另外創造一些可能會用到的日期或時間變數：

-   `date`: 日期 `yyyy-mm-dd`
-   `hour`: 每小時一筆資料 `00`, `01`, `02`, ..., `22`, `23`
-   `year`: 年份
-   `month`: 月份
-   `weekday`: 星期幾
-   `dailyType`: 雖然有假日與工作日變數，但是否有可能同時是不是假日也不是工作日的日期？

``` r
# date: yyyy-mm-dd
train$date <- substr(train$datetime, start = 1, stop = 10) %>% as.Date(.)
# hour
train$hour <- substr(train$datetime, start = 12, stop = 13) %>% factor(., levels = c("00", "01", "02", "03", "04", "05", "06", "07", "08", "09", "10", "11", "12", "13", "14", "15", "16", "17", "18", "19", "20", "21", "22", "23"))
# year
train$year <- year(train$date) %>% as.factor(.)
# month
train$month <- month(train$date) %>% as.factor(.)
# weekday
train$weekday <- strftime(train$date, format = "%w") %>% as.factor(.)
# dailyType
train$dailyType <- 0
train$dailyType[train$holiday == 0 & train$workingday == 0] <- 1
train$dailyType[train$holiday == 0 & train$workingday == 1] <- 2
train$dailyType[train$holiday == 1] <- 3
train$dailyType <- train$dailyType %>% as.factor(.)
```

為特徵設定合理的變數型態
------------------------

經過了特徵工程之後，首先先觀察我們現在擁有哪些特徵變數

``` r
names(train)
```

    ##  [1] "datetime"   "season"     "holiday"    "workingday" "weather"   
    ##  [6] "temp"       "atemp"      "humidity"   "windspeed"  "casual"    
    ## [11] "registered" "count"      "date"       "hour"       "year"      
    ## [16] "month"      "weekday"    "dailyType"

其中：

-   類別型變數： `season`, `holiday`, `workingday`, `weather`, `hour`, `year`, `month`, `weekday`, `dailyType`
-   連續型變數： `temp`, `atemp`, `humidity`, `windspeed`
-   反應變數：`casual`, `registered`, `count`

因此我們依照上述型態改變一下資料集內容的型態

``` r
# feature setup
## season
train$season <- ifelse(test = train$season == 1, 
                       yes = "spring", 
                       no = ifelse(test = train$season == 2, 
                                   yes = "summer", 
                                   no = ifelse(test = train$season == 3, 
                                               yes = "fall",
                                               no = "winter")))
## weather
train$weather <- ifelse(test = train$weather == 1, 
                       yes = "sunny", 
                       no = ifelse(test = train$weather == 2, 
                                   yes = "cloudy", 
                                   no = ifelse(test = train$weather == 3, 
                                               yes = "rainy",
                                               no = "extremely")))
# type
factorType <- c("season", "holiday", "workingday", "weather", "hour", "year", "month", "weekday", "dailyType")
numericType <- c("temp", "atemp", "humidity", "windspeed")
integerType <- c("casual", "registered", "count")
# check
train[, factorType] <- lapply(train[, factorType], as.factor)
train[, numericType] <- lapply(train[, numericType], as.numeric)
train[, integerType] <- lapply(train[, integerType], as.integer)
# order
train <- train %>% select(., date, hour, year, month, weekday, season, holiday, workingday, dailyType, weather, temp, atemp, humidity, windspeed, casual, registered, count)

# show 
str(train)
```

    ## 'data.frame':    10886 obs. of  17 variables:
    ##  $ date      : Date, format: "2011-01-01" "2011-01-01" ...
    ##  $ hour      : Factor w/ 24 levels "00","01","02",..: 1 2 3 4 5 6 7 8 9 10 ...
    ##  $ year      : Factor w/ 2 levels "2011","2012": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ month     : Factor w/ 12 levels "1","2","3","4",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ weekday   : Factor w/ 7 levels "0","1","2","3",..: 7 7 7 7 7 7 7 7 7 7 ...
    ##  $ season    : Factor w/ 4 levels "fall","spring",..: 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ holiday   : Factor w/ 2 levels "0","1": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ workingday: Factor w/ 2 levels "0","1": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ dailyType : Factor w/ 3 levels "1","2","3": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ weather   : Factor w/ 4 levels "cloudy","extremely",..: 4 4 4 4 4 1 4 4 4 4 ...
    ##  $ temp      : num  9.84 9.02 9.02 9.84 9.84 ...
    ##  $ atemp     : num  14.4 13.6 13.6 14.4 14.4 ...
    ##  $ humidity  : num  81 80 80 75 75 75 80 86 75 76 ...
    ##  $ windspeed : num  0 0 0 0 0 ...
    ##  $ casual    : int  3 8 5 3 0 0 2 1 1 8 ...
    ##  $ registered: int  13 32 27 10 1 1 0 2 7 6 ...
    ##  $ count     : int  16 40 32 13 1 1 2 3 8 14 ...

觀察反應變數
------------

我們的反應變數有三個，分別是 `casual`, `registered`以及`count`，其中 `count` 是 `casual`和`registered`的加總。 (ie.: , `count`=`casual`+`registered`)

通常我們在後面建模的部份都會先用簡單的模型來建立一個預測的**基準**，往後持續測試的模型或是調整都會以勝過**基準**為基本要求。這個題目最後的目的是預測 `count`，所以本質上這是一個屬於**迴歸(*Regression*)**的問題，因此我們會希望原始資料上的反應值應該要接近常態分佈，所以第一件事情想先觀察反應值是否為常態或是偏態。

-   由左圖可以看出我們的 `count` 資料是屬於右偏(*skew to right*)的，因此在後面建模的部份，我們可以考慮對其作轉換 (使用`box-cox`或是直接取 `log`)，希望讓它變成常態分佈。
-   右圖為取`log`之後的分佈圖，可以看到經過`log`轉換後還是沒有讓`count`的分佈達到常態，這在真實資料上是常見的。
-   等到要建立模型的時候，再測試有沒有其他的轉換函數有機會讓`count`的分佈達成常態。

``` r
pic_density_of_count <- ggplot(data = train, mapping = aes(x = count)) + 
  geom_histogram(mapping = aes(y = ..density..), fill = "blue") +
  geom_density(colour = "red") +
  theme_light() +
  theme(plot.background = element_blank()) +
  labs(title = "the density of response variables: count",
       x = "count",
       y = "density")

pic_density_of_log_count <- ggplot(data = train, mapping = aes(x = log(count + 1))) + 
  geom_histogram(mapping = aes(y = ..density..), fill = "blue") +
  geom_density(colour = "red") +
  theme_light() +
  theme(plot.background = element_blank()) +
  labs(title = "the density of response variables: log(count)",
       x = "log(count)",
       y = "density")


multiplot(pic_density_of_count, pic_density_of_log_count, cols = 2)
```

![unnamed-chunk-8-1](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-8-1.png)

觀察類別變數與反應變數的關係
----------------------------

接下來我們來觀察一下各個類別型變數與 `count` 的關係，而這邊我們先觀察**非日期相關**的變數，如：`season`, `holiday`, `workingday`, `weather`, `dailyType`等特徵。

-   不管是哪一種分類，都可以看到 `2012`年相對於 `2011` 年，租賃數有上升的趨勢。
    -   若要確認是否有顯著的上升，可以做一下統計檢定。
-   直接從 `count`的盒狀圖來看，有不少的離群值存在。
-   不同天氣型態有不同的租賃數表現，晴天的租賃數明顯大於雨天，而極端天氣在整個資料集裡只有一筆紀錄，所以我們暫不考慮它與其他天氣型態的比較。
-   不同季節也有明顯的差異，春季相較於其他季節租賃數偏低。
-   至於假日、工作日等型態，肉眼看不出明顯的差異，可以透過檢定判定。

``` r
# count
pic_box_count <- ggplot(data = train) +
  geom_boxplot(mapping = aes(x = year, y = count, fill = year)) +
  theme_light() +
  theme(plot.background = element_blank(),
        legend.position = "bottom") +
  labs(title = "",
       x = "year",
       y = "count")
# season
pic_box_season <- ggplot(data = train) +
  geom_boxplot(mapping = aes(x = year, y = count, fill = season)) +
  theme_light() +
  theme(plot.background = element_blank(),
        legend.position = "bottom") +
  labs(title = "",
       x = "season",
       y = "count")
# weather
pic_box_weather <- ggplot(data = train) +
  geom_boxplot(mapping = aes(x = year, y = count, fill = weather)) +
  theme_light() +
  theme(plot.background = element_blank(),
        legend.position = "bottom") +
  labs(title = "",
       x = "weather",
       y = "count")

# holiday
pic_box_holiday <- ggplot(data = train) +
  geom_boxplot(mapping = aes(x = year, y = count, fill = holiday)) +
  theme_light() +
  theme(plot.background = element_blank()) +
  labs(title = "",
       x = "holiday",
       y = "count")
# workingday
pic_box_workingday <- ggplot(data = train) +
  geom_boxplot(mapping = aes(x = year, y = count, fill = workingday)) +
  theme_light() +
  theme(plot.background = element_blank()) +
  labs(title = "",
       x = "workingday",
       y = "count")
# dailyType
pic_box_dailyType <- ggplot(data = train) +
  geom_boxplot(mapping = aes(x = year, y = count, fill = dailyType)) +
  theme_light() +
  theme(plot.background = element_blank()) +
  labs(title = "",
       x = "dailyType",
       y = "count")

# boxplot_one
layout_one <- matrix(c(1, 2, 3, 3), nrow = 2, byrow = TRUE)
multiplot(pic_box_count, pic_box_weather, pic_box_season, layout = layout_one)
```

![unnamed-chunk-9-1](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-9-1.png)

``` r
# boxplot_two
multiplot(pic_box_holiday, pic_box_workingday, pic_box_dailyType, cols = 1)
```

![unnamed-chunk-9-2](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-9-2.png)

觀察連續變數與反應變數的關係
----------------------------

接著，對於 `temp`, `atemp`, `humidity`, `windspeed`這些連續型變數，我們利用相關係數來了解各變數與`count`之間的關係。

-   `temp`和 `atemp` 符合直覺有極高的相關性，而後面建模時必須擇一，否則會有共線性的問題。
-   溫度(`temp`)與`count`有接近中度的相關性(*0.39*)。
-   濕度(`humidity`)與`count`有接近中度的相關性(*-0.31*)。
-   風速(`windspeed`)與所有變數都是低度相關。

``` r
ggpairs(data = train, 
        columns = c("temp", "atemp", "humidity", "windspeed", "count"),
        upper = list(continuous = wrap("cor", size = 8, alignPercent = 1)))
```

![unnamed-chunk-10-1](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-10-1.png)

其他角度的觀察
--------------

到目前為止，都是針對大類別的特徵做觀察，而我們最後的目的是預測**逐時**的單車租賃數，所以接下來要以**逐時**的角度加上不同類別的型態來做觀察，看看有什麼樣子的情況會讓租賃數有顯著的不同，這些資訊都會影響我們在後面建立模型的時候所使用的特徵工程，也就是會影響特徵的複雜程度。

因為在各個類別底下都有相當多的天數資料，為了簡單的看出是否有差異，我將會用各個類別來分組，並只取分組底下單車租賃數的**平均數**來作觀察。

用季節來分類發現，結果呼應前面類別變數與反應變數關係的觀察，`spring` 明顯的與其他季節的租賃數量有差距。

``` r
# by season
temp_season <- train %>% group_by(., hour, season) %>% summarise(., value = mean(count))
ggplot(data = temp_season) +
  geom_line(mapping = aes(x = hour, y = value, group = season, colour = season)) +
  theme_light() +
  theme(plot.background = element_blank(),
        legend.position = "bottom") +
  labs(title = "",
       x = "hour",
       y = "mean")
```

![unnamed-chunk-11-1](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-11-1.png)

由 `weekday`發現，星期六日與其他星期一二三四五的模式明顯不同，接著我們用`dailyType`來解釋可能更為明確：

-   前面特徵工程我們設定了 `dailyType` 變數，其中：
    -   `1` = 非假日且非工作日 (在平日的假日)
    -   `2` = 非假日但是工作日 (相當於平日)
    -   `3` = 完全的假日
-   由圖可以看出：
    -   類別 `2`在早上的 `6點至8點`以及下午的 `5點至7點` 明顯的是租賃的高峰，由於類別 `2`是上班日的類型，可以推測有不少人在上班日會選擇使用共享單車作為交通工具，移動至上班地點或是上課地點。
    -   而類別 `1`跟`3`的租賃高峰移動至下午時段，而原先在上班日的高峰時間反而鮮少人租賃，推測在假日時，大家選擇共享單車為休憩的交通工具。
        -   能夠用簡單的三個類別，分類出不同的使用行為，這對於後面建立模型可能會相當有幫助。

``` r
# by weekday
temp_weekday <- train %>% group_by(., hour, weekday) %>% summarise(., value = mean(count))
ggplot(data = temp_weekday) +
  geom_line(mapping = aes(x = hour, y = value, group = weekday, colour = weekday)) +
  theme_light() +
  theme(plot.background = element_blank(),
        legend.position = "bottom") +
  labs(title = "",
       x = "hour",
       y = "mean")
```

![unnamed-chunk-12-1](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-12-1.png)

``` r
# by dailyType
temp_dailyType <- train %>% group_by(., hour, dailyType) %>% summarise(., value = mean(count))
ggplot(data = temp_dailyType) +
  geom_line(mapping = aes(x = hour, y = value, group = dailyType, colour = dailyType)) +
  theme_light() +
  theme(plot.background = element_blank(),
        legend.position = "bottom") +
  labs(title = "",
       x = "hour",
       y = "mean")
```

![unnamed-chunk-12-2](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-12-2.png)

通篇的分析，我都只採用`count`來觀察租賃數與各種變數之間的關係，所以在最後，至少也想簡單的看一下，非註冊用戶(*casual*)與註冊用戶(*registered*)有無使用行為上的顯著差異。

-   由最後一張 `Count_Type` 的圖可以看出， 非註冊用戶與註冊用戶間有非常明顯不同的行為模式，因兩者呈現的線圖形狀明顯不同。
-   註冊用戶與非註冊用戶的**租賃次數**也有很大的差異。
-   推測有固定交通移動需求的上班族或是學生會偏好註冊，或許是有優惠或是基於方便性。

``` r
# by 註冊與否
temp_countType <- train %>% select(., hour, casual, registered) %>% group_by(., hour) %>% summarise(., mean_casual = mean(casual), mean_registered = mean(registered))
ggplot(data = temp_countType) +
  geom_line(mapping = aes(x = hour, y = mean_casual, group = 1, color = "casual")) +
  geom_line(mapping = aes(x = hour, y = mean_registered, group = 1, color = "registered")) +
  scale_color_manual(name = "Count_Type", values = c("casual" = "#33B31A80", "registered" = "#B3331A80")) +
  theme_light() +
  theme(plot.background = element_blank(),
        legend.position = "bottom") +
  labs(title = "",
       x = "hour",
       y = "mean")
```

![unnamed-chunk-13-1](https://jianhonglindst.github.io/assets/2018-02-24-eda-bike-sharing-demand-files/unnamed-chunk-13-1.png)

小結
----

-   租賃數的紀錄並不符合常態分佈，必須進行轉換。
-   `2011` 到 `2012` 租賃數有明顯的上升趨勢。
-   租賃數有離群值的存在。
-   進行建模時`溫度`與`體感溫度`必須擇一，否則可能會有共線性問題。
-   風速與租賃數低度相關。
-   在`season`, `weekday`, `dailyType`等類別個別分組下，租賃數都有不同的模式跑出，所以都是可幫助建模的特徵變數。
-   註冊用戶與非註冊用戶有明顯不同的模式，但基數有差距，可討論需否分開預測再合併計算總數。

以上，為 **Bike Sharing Demand** 這份資料進行的簡易但不嚴謹的探索性資料分析。
