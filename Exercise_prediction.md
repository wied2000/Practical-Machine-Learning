### 1 Excecutive Summary

One thing that people regularly do is quantify how much of a particular
activity they do, but they rarely quantify how well they do it. In this
project, your goal will be to use data from accelerometers on the belt,
forearm, arm, and dumbell of 6 participants.The goal of this assignment
is to predict the manner in which they did the exercise. Using
randomforest algorithm we create a prediction model with accuracy:

### 2 Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now
possible to collect a large amount of data about personal activity
relatively inexpensively. These type of devices are part of the
quantified self movement - a group of enthusiasts who take measurements
about themselves regularly to improve their health, to find patterns in
their behavior, or because they are tech geeks. One thing that people
regularly do is quantify how much of a particular activity they do, but
they rarely quantify how well they do it. In this project, our goal will
be to use data from accelerometers on the belt, forearm, arm, and
dumbell of 6 participants. They were asked to perform barbell lifts
correctly and incorrectly in 5 different way, thus 5 diferent classe
variables.

More information is available from the website here:
<http://web.archive.org/web/20161224072740/http:/groupware.les.inf.puc-rio.br/har>

### 3 Getting and Cleaning Data

#### 3.1 Load Data

We load the training and test data with defining that na.string
contains: (1)empty string (""), (2) "NA" and (3) "\#DIV/0!".

    pml.training <- read.csv("C:/Users/widya/Desktop/Coursera/Machine Learning/project/pml-training.csv",
                             header=T, na.strings=c("","#DIV/0!","NA"))
    pml.testing <- read.csv("C:/Users/widya/Desktop/Coursera/Machine Learning/project/pml-testing.csv",
                             header=T, na.strings=c("","#DIV/0!","NA"))

#### 3.2 Clean Data

The column with NA is identified as follows:

    pml.training.na <- apply(pml.training,2,function(f) sum(is.na(f)))
    pml.testing.na <- apply(pml.testing,2,function(f) sum(is.na(f)))
    length(pml.training.na[ pml.training.na > 1])

    ## [1] 100

    min(pml.training.na[ pml.training.na > 1])

    ## [1] 19216

Then we remove all those 100 columns with NA &gt; 19216  ()NA &ge; 20 for testing)

    pml.training <-  pml.training[,pml.training.na < 19216] 
    pml.testing <-  pml.testing[,pml.testing.na < 20] 

We also remove irrelevant column 1-7

    pml.training[1:2,c(1:7)]

    ##   X user_name raw_timestamp_part_1 raw_timestamp_part_2   cvtd_timestamp
    ## 1 1  carlitos           1323084231               788290 05/12/2011 11:23
    ## 2 2  carlitos           1323084231               808298 05/12/2011 11:23
    ##   new_window num_window
    ## 1         no         11
    ## 2         no         11

    pml.training <-  pml.training[,-c(1:7)] 
    pml.testing <-  pml.testing[,-c(1:7)] 
    dim(pml.training)

    ## [1] 19622    53

    dim(pml.testing)

    ## [1] 20 53

### 4 K-fold cross validation

#### 4.1 Load Library and Setting

    set.seed(0)
    library(caret)
    library(randomForest)
    library(party)


#### 4.2 Perform 10-fold cross validation as follows:

    k <- 10
    y <- c(1:nrow(pml.training))
    folds <- createFolds(y, k, list = TRUE, returnTrain = FALSE)
    obs <- list()
    pred <- list()
    for(i in 1:k){
      Indexes <- folds[[i]]
      testData <- pml.training[Indexes, ]
      trainData <- pml.training[-Indexes, ]
      #Use the test and train data partitions
      rf <- randomForest(classe ~ ., data=trainData)
      obs[[i]] <- testData$classe
      pred[[i]] <- predict(rf, testData)
    }

#### 4.3 Result

    accuracy = c()
    for(i in 1:k){
      conf_mat <- confusionMatrix(pred[[i]], obs[[i]])
      accuracy <- c(accuracy,conf_mat$overall[1])
    }

The accuracy range 

Max:

    ## [1] 0.9989806

Min:

    ## [1] 0.9949058

![](Excercise_prediction_files/figure-markdown_strict/plot_range-1.png)

### 5 Random Forest Algorithm for the Prediction

The 10-fold cross validation shows that random forest algorithm performs
veryy well with accuracy range of 0.9964 to 0.9969. So, we use as the
model for generating prediction for testing data.

    rf <- randomForest(classe ~ ., data=pml.training)
    varImpPlot(rf)

![](Excercise_prediction_files/figure-markdown_strict/prediction-1.png)

    prediction <- predict(rf, pml.testing)
    prediction 

    ##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
    ##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
    ## Levels: A B C D E
