---
title: Vote abstention
author: Felix MIL
date: '2022-04-10'
slug: vote-abstention
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2022-04-10T14:32:09+02:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


# Summary


# Package


```r
library(readxl)
library(janitor)
```

```
## Warning: le package 'janitor' a été compilé avec la version R 4.1.3
```

```
## 
## Attachement du package : 'janitor'
```

```
## Les objets suivants sont masqués depuis 'package:stats':
## 
##     chisq.test, fisher.test
```

```r
library(here)
```

```
## Warning: le package 'here' a été compilé avec la version R 4.1.3
```

```
## here() starts at D:/Programmation/personal-website
```

```r
library(tidyverse)
```

```
## -- Attaching packages --------------------------------------- tidyverse 1.3.1 --
```

```
## v ggplot2 3.3.5     v purrr   0.3.4
## v tibble  3.1.6     v dplyr   1.0.8
## v tidyr   1.1.4     v stringr 1.4.0
## v readr   2.1.2     v forcats 0.5.1
```

```
## Warning: le package 'readr' a été compilé avec la version R 4.1.3
```

```
## Warning: le package 'dplyr' a été compilé avec la version R 4.1.3
```

```
## -- Conflicts ------------------------------------------ tidyverse_conflicts() --
## x dplyr::filter() masks stats::filter()
## x dplyr::lag()    masks stats::lag()
```

```r
library(CARTElette)
```

```
## Afin de mieux connaitre les utilisateurs des packages COGugaison et CARTElette et de mieux repondre a vos besoins, merci de repondre a cette rapide enquete en ligne :
## https://antuki.github.io/2019/11/08/opinion_package/
```

```
## Registered S3 method overwritten by 'geojsonlint':
##   method         from 
##   print.location dplyr
```

```
## Afin de mieux connaitre les utilisateurs des packages COGugaison et CARTElette et de mieux repondre a vos besoins, merci de repondre a cette rapide enquete en ligne :
## https://antuki.github.io/2019/11/08/opinion_package/
```



# Data


```r
data <- 
  readxl::read_xlsx(here("content/post/2022-04-10-vote-abstention/10-avril-2022-taux-de-participation-premier-tour-12-h.xlsx"),
                  range = "B5:D101") %>%
  janitor::clean_names() %>%
  rename(dep_id = x1,
         dep_name = departement,
          taux = taux_participation_2022_1er_tour_12h)
```

```
## New names:
## * `` -> `...1`
```

```r
# data <-
data %>%
  mutate(dep_id = str_pad(dep_id, 2, pad = "0"))
```

```
## # A tibble: 96 x 3
##    dep_id dep_name                 taux
##    <chr>  <chr>                   <dbl>
##  1 01     Ain                      29.4
##  2 02     Aisne                    22.9
##  3 03     Allier                   29.1
##  4 04     Alpes-de-Haute-Provence  29.7
##  5 05     Hautes-Alpes             30.3
##  6 06     Alpes-Maritimes          25.3
##  7 07     Ardèche                  35.6
##  8 08     Ardennes                 24.0
##  9 09     Ariège                   30.1
## 10 10     Aube                     25.8
## # ... with 86 more rows
```


```r
dep_sf <- charger_carte(nivsupra = "DEP", geometrie_simplifiee = TRUE)
```

