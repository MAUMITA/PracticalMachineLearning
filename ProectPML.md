---
title: "Practical Machine Learning Course Project Writeup"
author: "Maumita Niyogi"
date: "Sunday, October 25, 2015"
output: pdf_document
keep_md: yes
cache: yes
---
##Summary
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, our goal is to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).


The goal of our project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. We may use any of the other variables to predict with.
We have to create a report describing 
1) how we built our model
2) how we used cross validation
3) what we think the expected out of sample error is
4) why we made the choices we did. 

##Data Preprocessing

```{r,results='hide'}
library(caret)
library(rpart)
library(rpart.plot)
library(randomForest)
library(corrplot)
```

The training data for this project are available here: 
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv

The test data are available here: 
https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv

Source: http://groupware.les.inf.puc-rio.br/har. 

###Read Data 
After downloading the data from the data source, we can read the two csv files into two data frames.
```{r}
trainRaw <- read.csv("pm1training.csv")
testRaw <- read.csv("pm1testing.csv")
dim(trainRaw)
dim(testRaw)
```
The training data set contains 19622 observations and 160 variables.
The testing data set contains 20 observations and 160 variables. 
The "classe" variable in the training set is the outcome to predict.

###Clean the data

```{r}
sum(complete.cases(trainRaw))
```
First, we remove columns that contain NA missing values.
```{r}
trainRaw <- trainRaw[, colSums(is.na(trainRaw)) == 0] 
testRaw <- testRaw[, colSums(is.na(testRaw)) == 0] 
```
Next, remove columns that do not contribute much to the accelerometer measurements.
```{r}
classe <- trainRaw$classe
trainRemove <- grepl("^X|timestamp|window", names(trainRaw))
trainRaw <- trainRaw[, !trainRemove]
trainCleaned <- trainRaw[, sapply(trainRaw, is.numeric)]
trainCleaned$classe <- classe
testRemove <- grepl("^X|timestamp|window", names(testRaw))
testRaw <- testRaw[, !testRemove]
testCleaned <- testRaw[, sapply(testRaw, is.numeric)]
dim(trainCleaned)
dim(testCleaned)
```
Now, the cleaned training data set contains 19622 observations and 53 variables.
The testing data set contains 20 observations and 53 variables.
The "classe" variable is still in the cleaned training set.

Then, we can split the cleaned training set into a pure training data set (70%) and a validation data set (30%). We will use the validation data set to conduct cross validation in future steps.
```{r}
set.seed(22519) # For reproducibile purpose
inTrain <- createDataPartition(trainCleaned$classe, p=0.70, list=F)
trainData <- trainCleaned[inTrain, ]
testData <- trainCleaned[-inTrain, ]
```
###Data Modeling
We fit a predictive model for activity recognition using Random Forest algorithm . We will use 5-fold cross validation when applying the algorithm.

```{r}
controlRf <- trainControl(method="cv", 5)
modelRf <- train(classe ~ ., data=trainData, method="rf", trControl=controlRf, ntree=250)
modelRf
```
Then, we estimate the performance of the model on the validation data set.
```{r}
predictRf <- predict(modelRf, testData)
confusionMatrix(testData$classe, predictRf)
accuracy <- postResample(predictRf, testData$classe)
accuracy
perf <- 1 - as.numeric(confusionMatrix(testData$classe, predictRf)$overall[1])
perf
```
So, the estimated accuracy of the model is 99.42% and the estimated out-of-sample error is 0.70%.

##Predicting for Test Data Set

Now, we apply the model to the original testing data set downloaded from the data source. We remove the problem_id column first.
```{r}
result <- predict(modelRf, testCleaned[, -length(names(testCleaned))])
result
```
##Appendix: Figures

Correlation Matrix Visualization
```{r}

corrPlot <- cor(trainData[, -length(names(trainData))])
corrplot(corrPlot, method="color")
```
Decision Tree Visualization
```{r}
treeModel <- rpart(classe ~ ., data=trainData, method="class")
prp(treeModel) # fast plot
```