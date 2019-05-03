---
title: "Observed Changes in Padma River from Landsat using Random Forest"
author: "Saad Tarik"
date: '2019-04-30'
comment: yes
disqus: true
contentCopyright: no
categories: []
description: ''
keywords: []
lastmod: '2019-04-30T05:07:36-04:00'
mathjax: yes
reward: no
slug: landsatriverchange
tags:
- R
- Landsat
- Padma
- Bangladesh
- Random Forest
autoCollapseToc: yes
toc: yes
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

# Train Data

I created training polygons for both 1989 and 2019 images. The training polygons have to two classes: River and No River so that we can identify the changes in the river course easily. 

```r
training_2019 <- shapefile("Train_2019.shp")
training_1989 <- shapefile("Train_1989.shp")
```
The next step is to rasterize the polygon objects. It creates a raster with the values from the corresponding training polygons. Then the Landsat image is masked with the training polygons and combined with the newly-created raster containing the classes from the previous step (using `rasterize`) in a stack. 
``` r
# 1989
classes_1989 <- rasterize(training_1989, all_1989_st_crop, field = "Class_ID")
names(classes_1989) <- "Classes"
mask_train_1989 <- mask(all_1989_st_crop, classes_1989)
# Combine the classes raster with the masked raster
mask_train_1989_class <- addLayer(mask_train_1989, classes_1989)

# 2019
classes_2019 <- rasterize(training_2019, all_2019_st_crop, field = "Class_ID")
names(classes_2019) <- "Classes"
mask_train_2019 <- mask(all_2019_st_crop, classes_2019)
# Combine the classes raster with the masked raster
mask_train_2019_class <- addLayer(mask_train_2019, classes_2019)
```
Then a `data.frame` is created with the training data which will be used in the `randomForest` function as an input.

``` r
# 1989
mask_train_1989_values <- getValues(mask_train_1989_class) # Get the values 
mask_train_1989_values <- na.omit(mask_train_1989_values) # Remove NAs
df.mask_train_1989_values <- data.frame(mask_train_1989_values) # Convert to data frame
df.mask_train_1989_values$Classes <- as.factor(df.mask_train_1989_values$Classes) # Convert classes to factor type

# 2019
mask_train_2019_values <- getValues(mask_train_2019_class) # Get the values 
mask_train_2019_values <- na.omit(mask_train_2019_values) # Remove NAs
df.mask_train_2019_values <- data.frame(mask_train_2019_values) # Convert to data frame
df.mask_train_2019_values$Classes <- as.factor(df.mask_train_2019_values$Classes) # Convert classes to factor type
```
# Run Random Forest
Now we have all the necessary inputs for the `randomForest` function. This function belongs to the package `randomForest`. It produces a model object as an output which can be used for the prediction in the next step.
``` r
library(randomForest)
model_rf_1989 <- randomForest(x = df.mask_train_1989_values[, c(1:6)],
                              y =df.mask_train_1989_values$Classes,
                              importance = TRUE)

colnames(model_rf_1989$confusion) <- c("No River", "River", "Class Error")
rownames(model_rf_1989$confusion) <- c("No River", "River")

model_rf_2019 <- randomForest(x = df.mask_train_2019_values[, c(1:7)],
                              y =df.mask_train_2019_values$Classes,
                              importance = TRUE)

colnames(model_rf_2019$confusion) <- c("No River", "River", "Class Error")
rownames(model_rf_2019$confusion) <- c("No River", "River")

# Prediction
predict_rf_1989 <- predict(all_1989_st_crop, model = model_rf_1989, na.rm = TRUE)
predict_rf_2019 <- predict(all_2019_st_crop, model = model_rf_2019, na.rm = TRUE)

# Visualilze
plot(predict_rf_2019, col = c("red", "blue"))
legend("bottomleft", legend = c("No River", "River"), fill = c("red", "blue"), bg = "white")
```
This figure shows the prediction from the year 2019 using random forest.
<img src="/post/2019-04-30-landsatriverchange_files/Screen Shot 2019-05-03 at 2.17.55 AM.png" alt="" width="80%"/>

To visualize the final result of the changes in river course from 1989 to 2019, we need to set the ‘No River’ class as `NA`. Then the river classes from 1989 and 2019 are plotted on top of the other using the package `quickPlot`.

``` r
# Set No River as NA export as polygon
predict_rf_1989[predict_rf_1989 == 1] <- NA
predict_rf_2019[predict_rf_2019 == 1] <- NA

# Combined Plot
library(quickPlot)
clearPlot()
Plot(predict_rf_1989, col = "red", legend = FALSE, new = TRUE, title = "")
Plot(predict_rf_2019, col = "blue", legend = FALSE, addTo = "predict_rf_1989", title = FALSE)
legend("bottomleft", legend = c("1989", "2019"), fill = c("red", "blue"), bg = "white")
```
<img src="/post/2019-04-30-landsatriverchange_files/Screen Shot 2019-05-03 at 2.20.13 AM.png" alt="1989 vs 2019" width="90%" height="90%"/>

# Further Readings and References
For more information, I would suggest the article published by NASA, [World of Change: Padma River](https://earthobservatory.nasa.gov/world-of-change/PadmaRiver) that includes references for relevant studies. It also includes a nice animation showing the change in the river course. I found these two tutorials ([Tutorial 1](https://www.earthdatascience.org/courses/earth-analytics/multispectral-remote-sensing-data/landsat-data-in-r-geotiff/) and [Tutorial 2](https://geoscripting-wur.github.io/AdvancedRasterAnalysis/)) that helped me to process the landsat data and running the random forest. 