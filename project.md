---
output: html_document
---
Personal Activity Prediction
========================================================
### Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

### Goal of this project

The goal of this project is to predict the manner in which they did the exercise. 

### Load the packages

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
library(gtools)
```

### Import train and test data from downloaded files

```r
# load the training data
train <- read.csv("pml-training.csv", header = T) 
# load the testing data
test <- read.csv("pml-testing.csv", header = T)
dim <- dim(train)
```

The training data contains 19622 observations and  160 variables. If we look at the data,  some variables are defined only for the case that  variable new_window = "no", while some variables are only defined for the case that variable newwindow = "yes". Basically, I want to seperate two cases, and remove reduct variables for each cases. Therefore, there will be easier to train the data. Since the newwindow for all test data equals to "no", here I only tried to find the model for this case.  

The first step of data cleaning is to subset the training data according to the variable new_window. The second step is to remove redundant variables. 

```r
# subset the training data according to new_window
train1 <- subset(train, new_window == "no")
# find the viables which is not empty or "NA" according to the first row of training data.
data <- train1[1,]
row  <- 0
for (i in 1:length(data)){
      if (!is.na(data[1,i]) & data[1,i]!= ""){
            row <- c(row, i)   
      }
      
}
row <- row[-1]
train1 <- train1[,row]
# remove first seventh variables since they are ids, names...
train1 <- train1[,8:60]
# apply the same precedure to test data
test1 <- test[,row]
test1 <- test1[,8:60]
```

Separate the training data into training data and crossvalidation. Use random forest method to train the data.

```r
invalid <- createDataPartition(train1$classe, p = 0.7)[[1]]
training <- train1[invalid,]
# crossvalidation data
crossVal <- train1[-invalid, ]
model <- train(classe ~., method = "rf", data = training, prox = T)
```

The accurary of the data is around 0.988. 


Apply the model to crossvalidation data and check the result.

```r
pre <- predict(model, cro)
sum(pre == crossVal$classe)/length(pre)
```

The accuracy for crossvalidation data is 0.992, which is close to the traing error.  

Apply the model to test data, and save the result into individual files.

```r
testResult <- predict(model, newdata = test1)
pml_write_files = function(x){
      n = length(x)
      for(i in 1:n){
            filename = paste0("problem_id_",i,".txt")
            write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
      }
}

pml_write_files(testResult)
```
