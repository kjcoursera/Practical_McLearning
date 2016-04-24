# Assignment: Prediction Assignment Writeup
April 22, 2016  

### Background 

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement – a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 

In this project, the goal is to build a model for the feature "classe" variable, discuss the model accuracy and apply to the test data set of twenty specific cases. 

The data is from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways.

### Data Preprocessing

Download the data

```r
# Download the files
download.file("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", destfile = "./pml-training.csv")
download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", destfile = "./pml-testing.csv")
```

Read the training data

```r
pmlTrain_raw <- read.csv("pml-training.csv",header = TRUE,na.strings = c("NA","NULL",""))
dim(pmlTrain_raw)
```

```
## [1] 19622   160
```

Removing NA's and the columns not necessary for processing

```r
# removing NA's
pmlTrain_raw <- pmlTrain_raw[,colSums(is.na(pmlTrain_raw))==0]

# removing unwanted columns
pmlTrain <- pmlTrain_raw[,-(1:7)]
```

Set the seed

```r
set.seed(12345)
```

We now split the dataset into a training dataset (70% of the observations) and a validation dataset (30% of the observations) using the caret function. 


```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
inTrain <- createDataPartition(y=pmlTrain$classe, p=0.7, list=FALSE)
training <- pmlTrain[inTrain,]
testing <- pmlTrain[-inTrain,]
dim(training)
```

```
## [1] 13737    53
```

```r
dim(testing)
```

```
## [1] 5885   53
```



Training with RANDOM FOREST :This is supervised learning classification problem. 


```r
ctrl <- trainControl(allowParallel = T, method = "cv",number = 5)
model.rf <- train(classe ~ ., data = training,model="rf",trControl = ctrl)
```

```
## Loading required package: randomForest
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```





```r
pred.rf <- predict(model.rf,newdata = testing)
confusionMatrix(testing$classe,pred.rf)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1674    0    0    0    0
##          B   11 1126    2    0    0
##          C    0   12 1012    2    0
##          D    0    0   31  933    0
##          E    0    0    0    2 1080
## 
## Overall Statistics
##                                           
##                Accuracy : 0.9898          
##                  95% CI : (0.9869, 0.9922)
##     No Information Rate : 0.2863          
##     P-Value [Acc > NIR] : < 2.2e-16       
##                                           
##                   Kappa : 0.9871          
##  Mcnemar's Test P-Value : NA              
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9935   0.9895   0.9684   0.9957   1.0000
## Specificity            1.0000   0.9973   0.9971   0.9937   0.9996
## Pos Pred Value         1.0000   0.9886   0.9864   0.9678   0.9982
## Neg Pred Value         0.9974   0.9975   0.9932   0.9992   1.0000
## Prevalence             0.2863   0.1934   0.1776   0.1592   0.1835
## Detection Rate         0.2845   0.1913   0.1720   0.1585   0.1835
## Detection Prevalence   0.2845   0.1935   0.1743   0.1638   0.1839
## Balanced Accuracy      0.9967   0.9934   0.9828   0.9947   0.9998
```

The estimated accuracy of the model is 98.9804588%

The most important variables in the model:


```r
varImp(model.rf)
```

```
## rf variable importance
## 
##   only 20 most important variables shown (out of 52)
## 
##                      Overall
## roll_belt             100.00
## yaw_belt               81.80
## magnet_dumbbell_z      70.42
## magnet_dumbbell_y      66.56
## pitch_forearm          61.84
## pitch_belt             61.48
## magnet_dumbbell_x      52.84
## roll_forearm           48.95
## accel_dumbbell_y       46.22
## magnet_belt_z          46.18
## magnet_belt_y          45.25
## accel_belt_z           43.32
## roll_dumbbell          43.21
## accel_dumbbell_z       41.72
## roll_arm               35.73
## accel_forearm_x        31.29
## yaw_dumbbell           29.91
## accel_dumbbell_x       29.51
## total_accel_dumbbell   28.90
## accel_arm_x            28.33
```

### Predicting on Test Data set:
As can be seen from the confusion matrix the above model is very accurate. Now we will apply to the test dataset


```r
pmlTest <-   read.csv("pml-testing.csv",header = TRUE,na.strings = c("NA","NULL","")) 
pred.test <- predict(model.rf,newdata=pmlTest)
pred.test
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

