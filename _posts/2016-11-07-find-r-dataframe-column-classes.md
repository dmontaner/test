---
layout:   post
comments: true
share:    true
date:     2016-01-01
modified: 2017-02-02
title:   "Finding R data.frame clas for all columns"
excerpt: "some tricks to find..."
description: "How to figure out URLs internally used in some R packages"
tags: [R, DataAnalysis]
categories: articles
---

Data

Sin R

```
datos <- data.frame (my.integer =  1:10, my.numeric = .5:10, my.logical =  TRUE, my.factor  = "I am a Level")
```



```r
datos <- data.frame (my.integer =  1:10, my.numeric = .5:10, my.logical =  TRUE, my.factor  = "I am a Level")
```

cosa


```r
datos <- data.frame (my.integer =  1:10,
                     my.numeric = .5:10,
                     my.logical =  TRUE,
                     my.factor  = "I am a Level"
                     )
```

Genearall you can figure out the class of each column doing: 


```r
sapply (datos, class)
```

```
## my.integer my.numeric my.logical  my.factor 
##  "integer"  "numeric"  "logical"   "factor"
```

Notice that by default `data.frame` creates "factor" variables from _text strings_ you can
modify this patern setting `stringsAsFactors = FALSE` in the data.frame function.
Or direclty including the text into a new (unammed) column 


```r
datos[,'my.character'] <- "txt"
```

Same works for Dates


```r
datos[,'todat'] <-  as.Date ("2016-11-07")
```

and is nice to be able to have the result in a vector for instance to select columns acording to their clases 
select non character columns


```r
cols.to.use <- sapply (datos, class) != "character"
datos[,cols.to.use]
```

```
##    my.integer my.numeric my.logical    my.factor      todat
## 1           1        0.5       TRUE I am a Level 2016-11-07
## 2           2        1.5       TRUE I am a Level 2016-11-07
## 3           3        2.5       TRUE I am a Level 2016-11-07
## 4           4        3.5       TRUE I am a Level 2016-11-07
## 5           5        4.5       TRUE I am a Level 2016-11-07
## 6           6        5.5       TRUE I am a Level 2016-11-07
## 7           7        6.5       TRUE I am a Level 2016-11-07
## 8           8        7.5       TRUE I am a Level 2016-11-07
## 9           9        8.5       TRUE I am a Level 2016-11-07
## 10         10        9.5       TRUE I am a Level 2016-11-07
```

one class I use a lot and that breacks this patter are date time clases


```r
datos[,'my.time'] <- Sys.time ()
datos
```

```
##    my.integer my.numeric my.logical    my.factor my.character      todat
## 1           1        0.5       TRUE I am a Level          txt 2016-11-07
## 2           2        1.5       TRUE I am a Level          txt 2016-11-07
## 3           3        2.5       TRUE I am a Level          txt 2016-11-07
## 4           4        3.5       TRUE I am a Level          txt 2016-11-07
## 5           5        4.5       TRUE I am a Level          txt 2016-11-07
## 6           6        5.5       TRUE I am a Level          txt 2016-11-07
## 7           7        6.5       TRUE I am a Level          txt 2016-11-07
## 8           8        7.5       TRUE I am a Level          txt 2016-11-07
## 9           9        8.5       TRUE I am a Level          txt 2016-11-07
## 10         10        9.5       TRUE I am a Level          txt 2016-11-07
##                my.time
## 1  2016-11-07 12:35:10
## 2  2016-11-07 12:35:10
## 3  2016-11-07 12:35:10
## 4  2016-11-07 12:35:10
## 5  2016-11-07 12:35:10
## 6  2016-11-07 12:35:10
## 7  2016-11-07 12:35:10
## 8  2016-11-07 12:35:10
## 9  2016-11-07 12:35:10
## 10 2016-11-07 12:35:10
```

because


```r
class (Sys.time ())
```

```
## [1] "POSIXct" "POSIXt"
```

returns two values:
And thus sapply cannot _simplify_ the resulting list


```r
sapply (datos, class)
```

```
## $my.integer
## [1] "integer"
## 
## $my.numeric
## [1] "numeric"
## 
## $my.logical
## [1] "logical"
## 
## $my.factor
## [1] "factor"
## 
## $my.character
## [1] "character"
## 
## $todat
## [1] "Date"
## 
## $my.time
## [1] "POSIXct" "POSIXt"
```

In this cases or if I am programing a library or a general function.
I usually use a double *apply to get a nice _simplified_ vector:


```r
sapply (lapply (datos, class), "[", 1)
```

```
##   my.integer   my.numeric   my.logical    my.factor my.character 
##    "integer"    "numeric"    "logical"     "factor"  "character" 
##        todat      my.time 
##       "Date"    "POSIXct"
```

thus, the _internal_ lapply gets the class list and the external 'sapply' retrieves just the first component of each of the vectos in the list
ie the "POSIXct" form the `c ("POSIXct" "POSIXt")`
and I cann keep selecting my columns as before 


```r
sapply (lapply (datos, class), "[", 1) ==  "POSIXct" 
```

```
##   my.integer   my.numeric   my.logical    my.factor my.character 
##        FALSE        FALSE        FALSE        FALSE        FALSE 
##        todat      my.time 
##        FALSE         TRUE
```

