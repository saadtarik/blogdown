---
title: Observed Changes in Padma River from Landsat using Random Forest
author: Saad Tarik
date: '2019-04-30'
slug: landsatriverchange
categories: []
tags: [R, Landsat, Padma, Bangladesh, Random Forest]
lastmod: '2019-04-30T05:07:36-04:00'
keywords: []
description: ''
comment: yes
toc: yes
autoCollapseToc: yes
contentCopyright: no
reward: no
mathjax: yes
---

<!--more-->
Rivers play an important part in the economy of Bangladesh. Millions of people rely on river for their daily activities and the eroding nature of rivers has already caused a lot of sufferings. In 2013, I finished a class project where I used decision tree classification to identify the changes in river course of the [Padma River](https://en.wikipedia.org/wiki/Padma_River) in Bangladesh. The river changed its course over time which is captured by NASA's [Landsat](https://landsat.gsfc.nasa.gov/) missions. Later, NASA published a story titled [World of Change: Padma River](https://earthobservatory.nasa.gov/world-of-change/PadmaRiver) visualizing the changes from 1988 to 2018. Here, I am reproducing the class project using R from 1989 to 2019 using [Random Forest](https://en.wikipedia.org/wiki/Random_forest).

# Data Used
I used Landsat data from 1989 and 2019 to document the changes. For simplicity, I selected the images that are free from cloud, at least at the location of the river. I clipped the study area with the river using shapefile that I created in QGIS. 

# Load Raw Data
I stored the raw `.tif` files from Landsat in a raster stack  as `.rda` after clipping the study area only. Then loaded data is shown as a composite in the following figure.
```r
load("Raw_Data/landsat.rda")
R <- 5
G <- 4
B <- 3

library(raster)
plotRGB(all_2019_st_crop, r = R, g = G, b = B, stretch = "lin", axes = TRUE,
        main = paste("Landsat composite w/ Bands", R,",", G,", and", B, "in 2019"))
box(col = "white")
```
<img src="/post/2019-04-30-landsatriverchange_files/Landsat2019_543.png" alt="Landsat 2019 543" height="300px"/>
