---
title: "Practical Machine Learning Project"
author: "Deepak Trivedi"
date: "February 7, 2016"
output: html_document
---

## Install packages

```{r, cache=TRUE,results="hide"}
#install.packages("caret")
require(caret)
#install.packages("rpart")
require(rpart)
#install.packages("rpart.plot")
require(rpart.plot)
#install.packages("e1071")
require(e1071)
#install.packages("randomForest")
require(randomForest)

```

## Initialize analysis and clean up data
Set seed for the project
```{r, cache=TRUE}
set.seed(1984)
```

Load data into R
```{r, cache=TRUE}
# Loading the training data set into my R session replacing all missing with "NA"
trnset <- read.csv("https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", na.strings=c("","NA","#DIV/0!"))

# Loading the testing data set 
tstset <- read.csv('https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv', na.strings=c("","NA","#DIV/0!"))
```

Look at all the column names. I have not echoed it here because the list is long.

```{r, cache=TRUE,results="hide"}
names(trnset)
```

We need to remove certain columns from the data set, like time, serial number etc.

```{r, cache=TRUE}
trnset   <-trnset[,-c(1:7)]
tstset <-tstset[,-c(1:7)]
```

Next, we remove columns with no data, since these will not help with our predictions in any way.

```{r, cache=TRUE}
trnset<-trnset[,colSums(is.na(trnset)) == 0]
tstset <-tstset[,colSums(is.na(tstset)) == 0]
```

This leaves us with 53 columns. The algorithm needs to predict the output variable classe, which describes the manner in which the dumbbell was lifted: (Class A), throwing the elbows to the front (Class B), lifting the dumbbell only halfway (Class C), lowering the dumbbell only halfway (Class D) and throwing the hips to the front (Class E).


Using the trnset, let's make a cross-validation set. 

```{r, cache=TRUE}
ssindex <- createDataPartition(y=trnset$classe, p=0.80, list=FALSE)
trnsub <- trnset[ssindex, ] 
tstsub <- trnset[-ssindex, ]
```

Looking at the heads of the training and test set. Not echoing here since it is long.
```{r, cache=TRUE,results="hide"}
head(trnsub)
head(tstsub)
```

Check the distribution of data in the various sets
```{r, cache=TRUE}
summary(trnset$classe)
summary(tstsub$classe)
summary(trnsub$classe)
```

In all three datasets, all the five classes (A,B,C,D,E) are fairly represented. 

## Decision Tree Model

This is the first model to try. We develop a decision tree and look at its performance. 


```{r, cache=TRUE}
dtreemodel <- rpart(classe ~ . , data=trnsub, method="class")
dtreeprediction <- predict(dtreemodel, tstsub, type = "class")
```

Plot the outcome of this model:
```{r, cache=TRUE}
rpart.plot(dtreemodel, main="Decision Tree Model",extra=1)
```

How good is this model? Let's look at the confusion matrix:

```{r, cache=TRUE}
cm1<-confusionMatrix(dtreeprediction, tstsub$classe)
cm1
```

## Random forest model

The decision tree model doesn't look very accurate. Next we try a random forest model. 

```{r, cache=TRUE}
rfmodel <- randomForest(classe ~ . , data=trnsub, method="class")
rfprediction <- predict(rfmodel, tstsub, type = "class")
```

How good is this model? Let's look at the confusion matrix
```{r, cache=TRUE}
cm2<-confusionMatrix(rfprediction, tstsub$classe)
cm2
```

Random forest model (Accuracy ~ ```r cm2$overall[1]```) is much better than the decision tree model (Accuracy ~ ```r cm1$overall[1]```.)

Expected out of sample error for the random forest model is ```r 1-cm2$overall[1]```. This looks pretty good. So, we will use this model to make predictions on the test data. 


##Predictions on test data

Use the predict function with this model and then look at the answers. 

```{r, cache=TRUE}
assignment_answer <- predict(rfmodel, tstset, type="class")
assignment_answer
```

## Conclusion

For this type of data, the random forest model seems to be better at predicting the outcome. The predictions listed above were used to answer the quiz questions, and all responses were found to be correct! 
