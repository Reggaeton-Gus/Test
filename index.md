

```r
---
title: "Prediction with repeated k-fold Cross Validation"
author: "Gustavo Escobar"
date: "3 September 2017"
output: html_document
---
```

```
## Error: <text>:8:0: unexpected end of input
## 6: ---
## 7: 
##   ^
```


## Introduction
Human Activity Recognition (HAR) is an increasing popular research area in pervasive computing, especially for the development of context-aware systems. Potential applications for HAR are eldercare, healthcare, and sports. 

One such application is using devices such as Jawboe Up, Nike FuelBand and Fitbit; where it is possible to collect a large amount of data about personal activities.  The data collected could be used to obtain statistics about how much of different activities people do or to quantify how well they do it. 

In this project, data collected from accelerometers on the belt, forearm, arm and dumbell of 6 participatns will be used to determine if an excercise was carried out in five different ways. Information regarding the experimental set-up can be find here.

A repeated k-fold Cross Validation technique will be employed to estimate Naive Bayes on the training set.

## Data
The data has already been divided into a training set and a test set. The sets can be downloaded as follows:


```r
URLTrain = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"

URLTest = "https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"

training = read.csv(url(URLTrain), na.strings = c("NA", "#DIV/0!", ""))
testing = read.csv(url(URLTest), na.strings = c("NA", "#DIV/0!", ""))
```

## Exploratory Analysis

We carry out a short Expolratory Analysis, to try to find out what sort of variables to ues for the model.


```r
no_training = dim(training)
no_testing = dim(testing)
lastColum_train = names(training[160])
lastColum_test = names(testing[160])
```

The training set has 19622 observations,  whereas the testing set has 20. Both of them have 160 variables. However, the variable "classe" only exists in the training data set. The testing data set has a variable called "problem_id"", instead.



```r
no_NA = sum(is.na(testing))
```

We also see that there are 2000 `NA` values.

The first seven variables contain data that will not be helpful in predictions such as index (x), user_name, etc.


```r
names(training [1:10])
```

```
##  [1] "X"                    "user_name"            "raw_timestamp_part_1"
##  [4] "raw_timestamp_part_2" "cvtd_timestamp"       "new_window"          
##  [7] "num_window"           "roll_belt"            "pitch_belt"          
## [10] "yaw_belt"
```



## Data Cleaning

It will be useful to delete the first seven columns of the data sets and delete the variables that have more than 20% `NA` values.


```r
testing = testing[,-c(1:7)]
training = training [,-c(1:7)]

f = function(x) { 
  sum(is.na(x)) < length(x) * 0.2
}

training = training[, vapply(training, f, logical(1)), drop = F]
testing = testing[, vapply(testing, f, logical(1)), drop = F]
```

The variable classe needs to be converted to a factor variable.

```r
training$classe= as.factor(training$classe)
```



## Repeated k-fold Cross Validation
We use the method of Repeated k-fold Cross Validation. We build the model using a random forest search. In this method, the data is split into k-folds and a random forest search is repeated a number of times. The final model accuracy is taken as the mean from the number of repeats. We use a 10-fold cross validation with 3 repeats to estimate Naive Bayes on the training set.


```r
train_control <- trainControl(method="repeatedcv", number=10, repeats=3)
model <- randomForest(classe~., data=training, trControl=train_control, method="class")
print(model)
```

```
## 
## Call:
##  randomForest(formula = classe ~ ., data = training, trControl = train_control,      method = "class") 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 7
## 
##         OOB estimate of  error rate: 0.3%
## Confusion matrix:
##      A    B    C    D    E  class.error
## A 5578    2    0    0    0 0.0003584229
## B   11 3782    4    0    0 0.0039504872
## C    0   12 3409    1    0 0.0037989480
## D    0    0   21 3193    2 0.0071517413
## E    0    0    1    4 3602 0.0013861935
```

The OOB estimate of error for the model is 0.3%.

## Model Predictions
The model is used to predict the class of exercise of the testing data set.


```r
answers = predict(model, testing)
answers
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```

## Conclusions
A three-repeats 10-fold cross validation scheme was devised to build a model using Human Activity Recognision data in order to predict the class of excercise from data collected using different sensors. Such cross validation technique provides a balanced approach to minimise the variance and bias of the model.
The model, bult with a random tree search, has an expected error of 0.3%.

End of assignment.
```

