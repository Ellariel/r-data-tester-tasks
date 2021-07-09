---
title: "Task1"
author: "Danila Valko"
date: "08 07 2021"
output: 
  html_document:
    keep_md: true
---



## Подтопления

### Грузим данные


```r
data <- read.csv(file = "mchs_floods_lena.csv", sep = ';', fileEncoding = "UTF-8")
head(data, 3)
```

```
##   X event_id             subject_name region_name  settlement_name object_name
## 1 1        3 Республика Саха (Якутия) Олекминский 1-й Нерюктяйинск        Лена
## 2 4        6 Республика Саха (Якутия)     Намский           Едейцы        Лена
## 3 5        7 Республика Саха (Якутия)     Намский            Арбын        Лена
##   flood_date house_count farm_count bridge_count road_count social_object_count
## 1 10.05.2013         201        241            0          0                   7
## 2 15.05.2013         201        241            0          0                   4
## 3 15.05.2013           2          2            0          0                   0
##   year flood_type
## 1 2013    Паводок
## 2 2013    Паводок
## 3 2013    Паводок
```

```r
tail(data, 3)
```

```
##     X event_id             subject_name region_name settlement_name object_name
## 16 57     2428 Республика Саха (Якутия)   Кобяйский        Кальвица        Лена
## 17 58     2429 Республика Саха (Якутия)   Кобяйский          Сангар        Лена
## 18 95     4934 Республика Саха (Якутия)   ГО Якутск          Якутск        Лена
##    flood_date house_count farm_count bridge_count road_count
## 16 20.05.2018          67         67            0          0
## 17 21.05.2018           1         15            0          0
## 18 12.05.2020           0         61            0          0
##    social_object_count year flood_type
## 16                   0 2018    паводок
## 17                   0 2018    паводок
## 18                   0 2020    паводок
```

### Общие замечания при визуальном осмотре (18 obs. x 14 vars.)
* По структуре файла: в зависимости от назначения файла нужно учесть, что кодировка UTF-8 без BOM не всегда может корректно считываться, то же касается одинокого LF в конце строки.
* По структуре данных:
  - Первая колонка(поле) не именована, возможно это номер строки. Если это действительно номер строки, то данные явно не полные, индексация должна быть без разрывов.
  - То же самое относительно разрывов в event_id: если это частичная выборка, то ok.
  - В колонках bridge_count, road_count, social_object_count по-видимому предполагаются целые числа, но данные представлены с плавающей точкой (0.0), для некоторого ПО это может стать проблемой. 
  - Относительно region_name и settlement_name: стоило бы добавить стандартные классификаторы, потому что наименование МО и ГО может различаться год от года.
  - В колонке flood_type различное написание одного и того же типа "Паводок" и "паводок"
  - В house_count и farm_count есть повторы для разных районов (см. event_id 3 и 6), нужно уточнять, возможно ли такое, что затопило одинаковое число домов с инервалом в пять дней.


### Посмотрим число уникальных значений по некоторым колонкам 


```r
print(paste("event_id     l:",length(data$event_id),"lu:",length(unique(data$event_id))))
```

```
## [1] "event_id     l: 18 lu: 18"
```

```r
print(paste("subject_name l:",length(data$subject_name),"lu:",length(unique(data$subject_name))))
```

```
## [1] "subject_name l: 18 lu: 1"
```

```r
print(paste("object_name  l:",length(data$object_name),"lu:",length(unique(data$object_name))))
```

```
## [1] "object_name  l: 18 lu: 1"
```

```r
print(paste("region_name  l:",length(data$region_name),"lu:",length(unique(data$region_name))))
```

```
## [1] "region_name  l: 18 lu: 7"
```

```r
print(paste("year         l:",length(data$year),"lu:",length(unique(data$year))))
```

```
## [1] "year         l: 18 lu: 3"
```

```r
print(paste("flood_type   l:",length(data$flood_type),"lu:",length(unique(data$flood_type))))
```

```
## [1] "flood_type   l: 18 lu: 2"
```

Всё в порядке, за исключением упомянутой выше flood_type

### Сводные статистики для пордка..и типы данных


```r
str(data)
```

```
## 'data.frame':	18 obs. of  14 variables:
##  $ X                  : int  1 4 5 6 7 8 9 10 49 50 ...
##  $ event_id           : int  3 6 7 8 9 10 11 12 2420 2421 ...
##  $ subject_name       : Factor w/ 1 level "Республика Саха (Якутия)": 1 1 1 1 1 1 1 1 1 1 ...
##  $ region_name        : Factor w/ 7 levels "ГО Якутск","Кобяйский",..: 5 4 4 1 1 1 6 6 7 7 ...
##  $ settlement_name    : Factor w/ 16 levels "1-й Нерюктяйинск",..: 1 4 2 16 12 6 7 3 8 14 ...
##  $ object_name        : Factor w/ 1 level "Лена": 1 1 1 1 1 1 1 1 1 1 ...
##  $ flood_date         : Factor w/ 8 levels "10.05.2013","12.05.2020",..: 1 3 3 3 3 3 3 3 4 4 ...
##  $ house_count        : int  201 201 2 39 8 54 82 43 11 0 ...
##  $ farm_count         : int  241 241 2 39 8 54 96 43 63 23 ...
##  $ bridge_count       : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ road_count         : num  0 0 0 0 0 0 0 0 0 0 ...
##  $ social_object_count: num  7 4 0 0 0 0 0 0 0 0 ...
##  $ year               : int  2013 2013 2013 2013 2013 2013 2013 2013 2018 2018 ...
##  $ flood_type         : Factor w/ 2 levels "паводок","Паводок": 2 2 2 2 2 2 2 2 1 1 ...
```

```r
summary(data)
```

```
##        X            event_id                         subject_name
##  Min.   : 1.00   Min.   :   3.00   Республика Саха (Якутия):18   
##  1st Qu.: 7.25   1st Qu.:   9.25                                 
##  Median :49.50   Median :2420.50                                 
##  Mean   :34.72   Mean   :1489.94                                 
##  3rd Qu.:53.75   3rd Qu.:2424.75                                 
##  Max.   :95.00   Max.   :4934.00                                 
##                                                                  
##               region_name         settlement_name object_name      flood_date
##  ГО Якутск          :5    Арбын           : 2     Лена:18     15.05.2013:7   
##  Кобяйский          :2    Якутск          : 2                 16.05.2018:3   
##  Мегино-Кангаласский:2    1-й Нерюктяйинск: 1                 17.05.2018:3   
##  Намский            :3    Ары-Тит         : 1                 10.05.2013:1   
##  Олекминский        :1    Едейцы          : 1                 12.05.2020:1   
##  Усть-Алданский     :3    Кальвица        : 1                 19.05.2018:1   
##  Хангаласский       :2    (Other)         :10                 (Other)   :2   
##   house_count       farm_count      bridge_count   road_count
##  Min.   :  0.00   Min.   :  2.00   Min.   :0     Min.   :0   
##  1st Qu.:  2.25   1st Qu.: 23.25   1st Qu.:0     1st Qu.:0   
##  Median : 11.50   Median : 41.00   Median :0     Median :0   
##  Mean   : 43.22   Mean   : 62.61   Mean   :0     Mean   :0   
##  3rd Qu.: 53.25   3rd Qu.: 66.00   3rd Qu.:0     3rd Qu.:0   
##  Max.   :201.00   Max.   :241.00   Max.   :0     Max.   :0   
##                                                              
##  social_object_count      year        flood_type
##  Min.   :0.0000      Min.   :2013   паводок:10  
##  1st Qu.:0.0000      1st Qu.:2013   Паводок: 8  
##  Median :0.0000      Median :2018               
##  Mean   :0.6111      Mean   :2016               
##  3rd Qu.:0.0000      3rd Qu.:2018               
##  Max.   :7.0000      Max.   :2020               
## 
```




.
