---
title: "Economic and Health Effects of Severe Weather in the US"
output: html_document
---
# Economic and Health Effects of Severe Weather in the US

## Synopsis

In this study we aim to determine which categories of severe weather events caused the most damage. We define damage in two ways - population health damage and economic damage - and then determine which types of events caused the most total damage. The data used are from the NOAA Storm Database from the years of 1950 to 2011. After cleaning our datasets, we determine that tornadoes caused the most population health damage via fatalities and injuries, and that floods caused the most monetary damage during this time period.

## Data Processing

First we read in the data. We can read it directly from the compressed ".bz2" archive file. We will also do some pre-processing. We change the event type (EVTYPE) variable to a factor variable. 


```r
storms <- read.csv("repdata-data-StormData.csv.bz2", stringsAsFactors = FALSE, na.strings = "")
storms$EVTYPE <- as.factor(storms$EVTYPE)
```

We will also need to do a little pre-processing of the variables containing property and crop damage data, which we will use in our analysis. Each value is broken down into a numerical value (e.g. PROPDMG) and a code denoting the order of magnitude (e.g. PROPDMGEXP) which could be "K" for thousands, "M" for millions, or "B" for billions of dollars. We will add a new column which contains the single numerical value for each of these variables, and call them PROPNUM and CROPNUM respectively. 


```r
storms$PROPNUM <- rep(0, each=nrow(storms))
storms$CROPNUM <- rep(0, each=nrow(storms))
storms$PROPDMGEXP <- as.factor(storms$PROPDMGEXP)
storms$CROPDMGEXP <- as.factor(storms$CROPDMGEXP)
summary(storms$PROPDMGEXP)
```

```
##      -      ?      +      0      1      2      3      4      5      6 
##      1      8      5    216     25     13      4      4     28      4 
##      7      8      B      h      H      K      m      M   NA's 
##      5      1     40      1      6 424665      7  11330 465934
```

```r
summary(storms$CROPDMGEXP)
```

```
##      ?      0      2      B      k      K      m      M   NA's 
##      7     19      1      9     21 281832      1   1994 618413
```

So we see that the majority of each type is a missing value. We would hope that most of these missing values have a zero value for the corresponding damage value column:


```r
sum(storms$PROPDMG[which(is.na(storms$PROPDMGEXP))] != 0)
```

```
## [1] 76
```

```r
sum(storms$CROPDMG[which(is.na(storms$CROPDMGEXP))] != 0)
```

```
## [1] 3
```

So there are only a very small number of missing codes with non-zero damage values relative to the size of our data set. We can safely ignore these values and only use rows for which we have a valid code:


```r
storms$PROPNUM[which(storms$PROPDMGEXP == 'K')] = 
    storms$PROPDMG[which(storms$PROPDMGEXP == 'K')] * 1000
storms$PROPNUM[which(storms$PROPDMGEXP %in% c('m','M'))] = 
    storms$PROPDMG[which(storms$PROPDMGEXP %in% c('m','M'))] * 1000000
storms$PROPNUM[which(storms$PROPDMGEXP == 'B')] = 
    storms$PROPDMG[which(storms$PROPDMGEXP == 'B')] * 1000000000

storms$CROPNUM[which(storms$CROPDMGEXP %in% c('k','K'))] = 
    storms$CROPDMG[which(storms$CROPDMGEXP %in% c('k','K'))] * 1000
storms$CROPNUM[which(storms$CROPDMGEXP %in% c('m','M'))] = 
    storms$CROPDMG[which(storms$CROPDMGEXP %in% c('m','M'))] * 1000000
storms$CROPNUM[which(storms$CROPDMGEXP == 'B')] = 
    storms$CROPDMG[which(storms$CROPDMGEXP == 'B')] * 1000000000
```

Finally, let us sum the property and crop damage columns and call the result TOTALDMG:


```r
storms$TOTALDMG <- storms$PROPNUM + storms$CROPNUM
```

Now that we have pre-processed our data, we can move on to our analysis.

## Results

### Effect of Various Events on Population Health

Let us consider how different types of severe weather events effect "population health", which will include the total number of injuries and fatalities caused by an event type. Let's look at the top 8 event types in terms of total fatalities caused:



```r
library(plyr)
stormSummary <- ddply(storms, .(EVTYPE), summarize, 
                      sumFatalities = sum(FATALITIES), sumInjuries = sum(INJURIES))
totalFatalityOrdered <- stormSummary[order(stormSummary$sumFatalities, decreasing=TRUE),]
head(totalFatalityOrdered, n = 8)
```

```
##             EVTYPE sumFatalities sumInjuries
## 834        TORNADO          5633       91346
## 130 EXCESSIVE HEAT          1903        6525
## 153    FLASH FLOOD           978        1777
## 275           HEAT           937        2100
## 464      LIGHTNING           816        5230
## 856      TSTM WIND           504        6957
## 170          FLOOD           470        6789
## 585    RIP CURRENT           368         232
```

We observe that tornadoes caused the most fatalities, by a wide margin. Now let's take a look at the top 8 in terms of injuries caused:



```r
totalInjuryOrdered <- stormSummary[order(stormSummary$sumInjuries, decreasing=TRUE),]
head(totalInjuryOrdered, n = 8)
```

```
##             EVTYPE sumFatalities sumInjuries
## 834        TORNADO          5633       91346
## 856      TSTM WIND           504        6957
## 170          FLOOD           470        6789
## 130 EXCESSIVE HEAT          1903        6525
## 464      LIGHTNING           816        5230
## 275           HEAT           937        2100
## 427      ICE STORM            89        1975
## 153    FLASH FLOOD           978        1777
```

Again, tornadoes caused the most damage by a wide margin. We can safely say that tornadoes were the most damaging to population health of all event types during the time period that the data was collected. We also notice that seven events appear in both top 8 lists. We conclude that these event types - Tornado, TSTM Wind, Flood, Excessive Heat, Lightning, Heat, and Flash Flood - were the most destructive (in terms of total damage, not average damage per event) to population health of all event types.

### Economic Effect of Various Events

Next we will examine the total amount of damage for each event type in terms of monetary, or economic, damage caused. We will look at the total damage, which includes property damage as well as crop damage. First let's create a data frame summarizing the total damage for each event type, and take note of the 


```r
dmgSummary <- ddply(storms, .(EVTYPE), summarize,
                    sumDmg = sum(TOTALDMG))
totalDmgOrdered <- dmgSummary[order(dmgSummary$sumDmg, decreasing=TRUE),]
head(totalDmgOrdered, n = 20)
```

```
##                        EVTYPE       sumDmg
## 170                     FLOOD 150319678250
## 411         HURRICANE/TYPHOON  71913712800
## 834                   TORNADO  57352113590
## 670               STORM SURGE  43323541000
## 244                      HAIL  18758221170
## 153               FLASH FLOOD  17562128610
## 95                    DROUGHT  15018672000
## 402                 HURRICANE  14610229010
## 590               RIVER FLOOD  10148404500
## 427                 ICE STORM   8967041310
## 848            TROPICAL STORM   8382236550
## 972              WINTER STORM   6715441250
## 359                 HIGH WIND   5908617560
## 957                  WILDFIRE   5060586800
## 856                 TSTM WIND   5038935790
## 671          STORM SURGE/TIDE   4642038000
## 760         THUNDERSTORM WIND   3897964190
## 409            HURRICANE OPAL   3191846000
## 955          WILD/FOREST FIRE   3108626330
## 298 HEAVY RAIN/SEVERE WEATHER   2500000000
```

So we see that the most damaging event type in terms of total monetary damage caused is FLOOD, and again by a wide margin. The other top 19 types are also included above, and we can confirm that these 20 types of events account for more than 95% of all monetary damage caused:


```r
sum(totalDmgOrdered$sumDmg[1:20])/sum(totalDmgOrdered$sumDmg)
```

```
## [1] 0.9580146
```
We also see that the top 10 most destructive event types account for more than 85% of all monetary damage caused:


```r
sum(totalDmgOrdered$sumDmg[1:10])/sum(totalDmgOrdered$sumDmg)
```

```
## [1] 0.856327
```

Finally, let us only consider these top 10 event types, and examine their relative magnitudes using a pie chart:


```r
top10 <- totalDmgOrdered[1:11,]
top10$EVTYPE <- as.character(top10$EVTYPE)
top10$EVTYPE[11] <- "OTHER"
top10$sumDmg[11] <- sum(totalDmgOrdered$sumDmg) - sum(top10$sumDmg[1:10])
pie(top10$sumDmg, labels = top10$EVTYPE, main="Breakdown of Damage Caused by Various Severe Weather Events")
```

![plot of chunk unnamed-chunk-10](figure/unnamed-chunk-10-1.png) 

In this figure we see that flood-related damage accounted for roughly one third of all monetary damage in this study, confirming graphically that floods were overall the most costly form of severe weather event. Hurricane/Typhoons, Tornadoes, and Storm Surges also contributed to sizable portions of the damage. After that there is a slight dip in the relative weight of the rest of the event types.
