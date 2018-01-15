# Nearest neighbours {#nearest-neighbours}

<!-- Matt -->

## Introduction
_k_-NN is by far the simplest method of supervised learning we will cover in this course. It is a non-parametric method that can be used for both classification (predicting class membership) and regression (estimating continuous variables). _k_-NN is categorized as instance based (memory based) learning, because all computation is deferred until classification. The most computationally demanding aspects of _k_-NN are finding neighbours and storing the entire learning set.

A simple _k_-NN classification rule (figure \@ref(fig:knnClassification)) would proceed as follows:

1. when presented with a new observation, find the _k_ closest samples in the learning set
2. predict the class by majority vote

<div class="figure" style="text-align: center">
<img src="images/knn_classification.svg" alt="Illustration of _k_-nn classification. In this example we have two classes: blue squares and red triangles. The green circle represents a test object. If k=3 (solid line circle) the test object is assigned to the red triangle class. If k=5 the test object is assigned to the blue square class.  By Antti Ajanki AnAj - Own work, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=2170282" width="75%" />
<p class="caption">(\#fig:knnClassification)Illustration of _k_-nn classification. In this example we have two classes: blue squares and red triangles. The green circle represents a test object. If k=3 (solid line circle) the test object is assigned to the red triangle class. If k=5 the test object is assigned to the blue square class.  By Antti Ajanki AnAj - Own work, CC BY-SA 3.0, https://commons.wikimedia.org/w/index.php?curid=2170282</p>
</div>

A basic implementation of _k_-NN regression would calculate the average of the numerical outcome of the _k_ nearest neighbours. 

The number of neighbours _k_ can have a considerable impact on the predictive performance of _k_-NN in both classification and regression. The optimal value of _k_ should be chosen using cross-validation.

Euclidean distance is the most widely used distance metric in _k_-nn, and will be used in the examples and exercises in this chapter. However, other distance metrics can be used.

**Euclidean distance:**
\begin{equation}
  distance\left(p,q\right)=\sqrt{\sum_{i=1}^{n} (p_i-q_i)^2}
  (\#eq:euclidean)
\end{equation}


<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/euclideanDistanceDiagram-1.png" alt="Euclidean distance." width="75%" />
<p class="caption">(\#fig:euclideanDistanceDiagram)Euclidean distance.</p>
</div>


## Classification: simulated data

A simulated data set will be used to demonstrate:

* bias-variance trade-off
* the knn function in R
* plotting decision boundaries
* choosing the optimum value of _k_

The dataset has been partitioned into training and test sets.

Load data

```r
load("data/example_binary_classification/bin_class_example.rda")
str(xtrain)
```

```
## 'data.frame':	400 obs. of  2 variables:
##  $ V1: num  -0.223 0.944 2.36 1.846 1.732 ...
##  $ V2: num  -1.153 -0.827 -0.128 2.014 -0.574 ...
```

```r
str(xtest)
```

```
## 'data.frame':	400 obs. of  2 variables:
##  $ V1: num  2.09 2.3 2.07 1.65 1.18 ...
##  $ V2: num  -1.009 1.0947 0.1644 0.3243 -0.0277 ...
```

```r
summary(as.factor(ytrain))
```

```
##   0   1 
## 200 200
```

```r
summary(as.factor(ytest))
```

```
##   0   1 
## 200 200
```


```r
library(ggplot2)
library(GGally)
library(RColorBrewer)
point_shapes <- c(15,17)
point_colours <- brewer.pal(3,"Dark2")
point_size = 2

ggplot(xtrain, aes(V1,V2)) + 
  geom_point(col=point_colours[ytrain+1], shape=point_shapes[ytrain+1], 
             size=point_size) + 
  ggtitle("train") +
  theme_bw() +
  theme(plot.title = element_text(size=25, face="bold"), axis.text=element_text(size=15),
        axis.title=element_text(size=20,face="bold"))

ggplot(xtest, aes(V1,V2)) + 
  geom_point(col=point_colours[ytest+1], shape=point_shapes[ytest+1], 
             size=point_size) + 
  ggtitle("test") +
  theme_bw() +
  theme(plot.title = element_text(size=25, face="bold"), axis.text=element_text(size=15),
        axis.title=element_text(size=20,face="bold"))
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/simDataBinClassTrainTest-1.png" alt="Scatterplots of the simulated training and test data sets that will be used in the demonstration of binary classification using _k_-nn" width="50%" /><img src="07-nearest-neighbours_files/figure-html/simDataBinClassTrainTest-2.png" alt="Scatterplots of the simulated training and test data sets that will be used in the demonstration of binary classification using _k_-nn" width="50%" />
<p class="caption">(\#fig:simDataBinClassTrainTest)Scatterplots of the simulated training and test data sets that will be used in the demonstration of binary classification using _k_-nn</p>
</div>


### knn function
For _k_-nn classification and regression we will use the **knn** function in the package **class**.

```r
library(class)
```

**Arguments to knn**

* ```train``` : matrix or data frame of training set cases.
* ```test``` : matrix or data frame of test set cases. A vector will be interpreted as a row vector for a single case.
* ```cl``` : factor of true classifications of training set
* ```k``` : number of neighbours considered.
* ```l``` : minimum vote for definite decision, otherwise doubt. (More precisely, less than k-l dissenting votes are allowed, even if k is increased by ties.)
* ```prob``` : If this is true, the proportion of the votes for the winning class are returned as attribute prob.
* ```use.all``` : controls handling of ties. If true, all distances equal to the kth largest are included. If false, a random selection of distances equal to the kth is chosen to use exactly k neighbours.

Let us perform _k_-nn on the training set with _k_=1. We will use the **confusionMatrix** function from the [caret](http://cran.r-project.org/web/packages/caret/index.html) package to summarize performance of the classifier.

```r
library(caret)
```

```
## Loading required package: lattice
```

```r
knn1train <- class::knn(train=xtrain, test=xtrain, cl=ytrain, k=1)
confusionMatrix(knn1train, ytrain)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   0   1
##          0 200   0
##          1   0 200
##                                      
##                Accuracy : 1          
##                  95% CI : (0.9908, 1)
##     No Information Rate : 0.5        
##     P-Value [Acc > NIR] : < 2.2e-16  
##                                      
##                   Kappa : 1          
##  Mcnemar's Test P-Value : NA         
##                                      
##             Sensitivity : 1.0        
##             Specificity : 1.0        
##          Pos Pred Value : 1.0        
##          Neg Pred Value : 1.0        
##              Prevalence : 0.5        
##          Detection Rate : 0.5        
##    Detection Prevalence : 0.5        
##       Balanced Accuracy : 1.0        
##                                      
##        'Positive' Class : 0          
## 
```
The classifier performs perfectly on the training set, because with _k_=1, each observation is being predicted by itself!
<!--
table(ytrain,knn1train)
cat("KNN prediction error for training set: ", 1-mean(as.numeric(as.vector(knn1train))==ytrain), "\n")
-->

Now let use the training set to predict on the test set.

```r
knn1test <- class::knn(train=xtrain, test=xtest, cl=ytrain, k=1)
confusionMatrix(knn1test, ytest)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   0   1
##          0 131  81
##          1  69 119
##                                           
##                Accuracy : 0.625           
##                  95% CI : (0.5755, 0.6726)
##     No Information Rate : 0.5             
##     P-Value [Acc > NIR] : 3.266e-07       
##                                           
##                   Kappa : 0.25            
##  Mcnemar's Test P-Value : 0.3691          
##                                           
##             Sensitivity : 0.6550          
##             Specificity : 0.5950          
##          Pos Pred Value : 0.6179          
##          Neg Pred Value : 0.6330          
##              Prevalence : 0.5000          
##          Detection Rate : 0.3275          
##    Detection Prevalence : 0.5300          
##       Balanced Accuracy : 0.6250          
##                                           
##        'Positive' Class : 0               
## 
```
Performance on the test set is not so good. This is an example of a classifier being over-fitted to the training set. 
<!--
table(ytest, knn1test)
cat("KNN prediction error for test set: ", 1-mean(as.numeric(as.vector(knn1test))==ytest), "\n")
-->

### Plotting decision boundaries
Since we have just two dimensions we can visualize the decision boundary generated by the _k_-nn classifier in a 2D scatterplot. Situations where your original data set contains only two variables will be rare, but it is not unusual to reduce a high-dimensional data set to just two dimensions using the methods that will be discussed in chapter \@ref(dimensionality-reduction). Therefore, knowing how to plot decision boundaries will potentially be helpful for many different datasets and classifiers.

Create a grid so we can predict across the full range of our variables V1 and V2.

```r
gridSize <- 150 
v1limits <- c(min(c(xtrain[,1],xtest[,1])),max(c(xtrain[,1],xtest[,1])))
tmpV1 <- seq(v1limits[1],v1limits[2],len=gridSize)
v2limits <- c(min(c(xtrain[,2],xtest[,2])),max(c(xtrain[,2],xtest[,2])))
tmpV2 <- seq(v2limits[1],v2limits[2],len=gridSize)
xgrid <- expand.grid(tmpV1,tmpV2)
names(xgrid) <- names(xtrain)
```

Predict values of all elements of grid.

```r
knn1grid <- class::knn(train=xtrain, test=xgrid, cl=ytrain, k=1)
V3 <- as.numeric(as.vector(knn1grid))
xgrid <- cbind(xgrid, V3)
```

Plot

```r
point_shapes <- c(15,17)
point_colours <- brewer.pal(3,"Dark2")
point_size = 2

ggplot(xgrid, aes(V1,V2)) +
  geom_point(col=point_colours[knn1grid], shape=16, size=0.3) +
  geom_point(data=xtrain, aes(V1,V2), col=point_colours[ytrain+1],
             shape=point_shapes[ytrain+1], size=point_size) +
  geom_contour(data=xgrid, aes(x=V1, y=V2, z=V3), breaks=0.5, col="grey30") +
  ggtitle("train") +
  theme_bw() +
  theme(plot.title = element_text(size=25, face="bold"), axis.text=element_text(size=15),
        axis.title=element_text(size=20,face="bold"))

ggplot(xgrid, aes(V1,V2)) +
  geom_point(col=point_colours[knn1grid], shape=16, size=0.3) +
  geom_point(data=xtest, aes(V1,V2), col=point_colours[ytest+1],
             shape=point_shapes[ytrain+1], size=point_size) +
  geom_contour(data=xgrid, aes(x=V1, y=V2, z=V3), breaks=0.5, col="grey30") +
  ggtitle("test") +
  theme_bw() +
  theme(plot.title = element_text(size=25, face="bold"), axis.text=element_text(size=15),
        axis.title=element_text(size=20,face="bold"))
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/simDataBinClassDecisionBoundaryK1-1.png" alt="Binary classification of the simulated training and test sets with _k_=1." width="50%" /><img src="07-nearest-neighbours_files/figure-html/simDataBinClassDecisionBoundaryK1-2.png" alt="Binary classification of the simulated training and test sets with _k_=1." width="50%" />
<p class="caption">(\#fig:simDataBinClassDecisionBoundaryK1)Binary classification of the simulated training and test sets with _k_=1.</p>
</div>

### Bias-variance tradeoff
The bias–variance tradeoff is the problem of simultaneously minimizing two sources of error that prevent supervised learning algorithms from generalizing beyond their training set:

* The bias is error from erroneous assumptions in the learning algorithm. High bias can cause an algorithm to miss the relevant relations between features and target outputs (underfitting).
* The variance is error from sensitivity to small fluctuations in the training set. High variance can cause an algorithm to model the random noise in the training data, rather than the intended outputs (overfitting).

To demonstrate this phenomenon, let us look at the performance of the _k_-nn classifier over a range of values of _k_.  First we will define a function to create a sequence of log spaced values. This is the **lseq** function from the [emdbook](https://cran.r-project.org/package=emdbook) package:

```r
lseq <- function(from, to, length.out) {
  exp(seq(log(from), log(to), length.out = length.out))
}
```

Get log spaced sequence of length 20, round and then remove any duplicates resulting from rounding.

```r
s <- unique(round(lseq(1,400,20)))
length(s)
```

```
## [1] 19
```


```r
train_error <- sapply(s, function(i){
  yhat <- knn(xtrain, xtrain, ytrain, i)
  return(1-mean(as.numeric(as.vector(yhat))==ytrain))
})

test_error <- sapply(s, function(i){
  yhat <- knn(xtrain, xtest, ytrain, i)
  return(1-mean(as.numeric(as.vector(yhat))==ytest))
})

k <- rep(s, 2)
set <- c(rep("train", length(s)), rep("test", length(s)))
error <- c(train_error, test_error)
misclass_errors <- data.frame(k, set, error)
```


```r
ggplot(misclass_errors, aes(x=k, y=error, group=set)) + 
  geom_line(aes(colour=set, linetype=set), size=1.5) +
  scale_x_log10() +
  ylab("Misclassification Errors") +
  theme_bw() +
  theme(legend.position = c(0.5, 0.25), legend.title=element_blank(),
        legend.text=element_text(size=12), 
        axis.title.x=element_text(face="italic", size=12))
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/misclassErrorsFunK-1.png" alt="Misclassification errors as a function of neighbourhood size." width="100%" />
<p class="caption">(\#fig:misclassErrorsFunK)Misclassification errors as a function of neighbourhood size.</p>
</div>
We see excessive variance (overfitting) at low values of _k_, and bias (underfitting) at high values of _k_.

### Choosing _k_

We will use the caret library. Caret provides a unified interface to a huge range of supervised learning packages in R. The design of its tools encourages best practice, especially in relation to cross-validation and testing. Additionally, it has automatic parallel processing built in, which is a significant advantage when dealing with large data sets.

```r
library(caret)
```

To take advantage of Caret's parallel processing functionality, we simply need to load the [doMC](http://cran.r-project.org/web/packages/doMC/index.html) package and register workers: 

```r
library(doMC)
```

```
## Loading required package: foreach
```

```
## Loading required package: iterators
```

```
## Loading required package: parallel
```

```r
registerDoMC()
```

To find out how many cores we have registered we can use:

```r
getDoParWorkers()
```

```
## [1] 2
```

The [caret](http://cran.r-project.org/web/packages/caret/index.html) function **train** is used to fit predictive models over different values of _k_. The function **trainControl** is used to specify a list of computational and resampling options, which will be passed to **train**. We will start by configuring our cross-validation procedure using **trainControl**.

We would like to make this demonstration reproducible and because we will be running the models in parallel, using the **set.seed** function alone is not sufficient. In addition to using **set.seed** we have to make use of the optional **seeds** argument to **trainControl**. We need to supply **seeds** with a list of integers that will be used to set the seed at each sampling iteration. The list is required to have a length of B+1, where B is the number of resamples. We will be repeating 10-fold cross-validation a total of ten times and so our list must have a length of 101. The first B elements of the list are required to be vectors of integers of length M, where M is the number of models being evaluated (in this case 19). The last element of the list only needs to be a single integer, which will be used for the final model.

First we generate our list of seeds.

```r
set.seed(42)
seeds <- vector(mode = "list", length = 101)
for(i in 1:100) seeds[[i]] <- sample.int(1000, 19)
seeds[[101]] <- sample.int(1000,1)
```

We can now use **trainControl** to create a list of computational options for resampling.

```r
tc <- trainControl(method="repeatedcv",
                   number = 10,
                   repeats = 10,
                   seeds = seeds)
```

There are two options for choosing the values of _k_ to be evaluated by the **train** function:

1. Pass a data.frame of values of _k_ to the **tuneGrid** argument of **train**.
2. Specify the number of different levels of _k_ using the **tuneLength** function and allow **train** to pick the actual values.

We will use the first option, so that we can try the values of _k_ we examined earlier. The vector of values of _k_ we created earlier should be converted into a data.frame.


```r
s <- data.frame(s)
names(s) <- "k"
```

We are now ready to run the cross-validation.

```r
knnFit <- train(xtrain, as.factor(ytrain), 
                method="knn",
                tuneGrid=s,
                trControl=tc)

knnFit
```

```
## k-Nearest Neighbors 
## 
## 400 samples
##   2 predictor
##   2 classes: '0', '1' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold, repeated 10 times) 
## Summary of sample sizes: 360, 360, 360, 360, 360, 360, ... 
## Resampling results across tuning parameters:
## 
##   k    Accuracy  Kappa 
##     1  0.63300   0.2660
##     2  0.63875   0.2775
##     3  0.67375   0.3475
##     4  0.67900   0.3580
##     5  0.69575   0.3915
##     7  0.71100   0.4220
##     9  0.71775   0.4355
##    12  0.71500   0.4300
##    17  0.72675   0.4535
##    23  0.73800   0.4760
##    32  0.73725   0.4745
##    44  0.73875   0.4775
##    60  0.74850   0.4970
##    83  0.75500   0.5100
##   113  0.73500   0.4700
##   155  0.72575   0.4515
##   213  0.70750   0.4150
##   292  0.68825   0.3765
##   400  0.51300   0.0260
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was k = 83.
```

**Cohen's Kappa:**
\begin{equation}
  Kappa = \frac{O-E}{1-E}
  (\#eq:kappa)
\end{equation}

where _O_ is the observed accuracy and _E_ is the expected accuracy based on the marginal totals of the confusion matrix. Cohen's Kappa takes values between -1 and 1; a value of zero indicates no agreement between the observed and predicted classes, while a value of one shows perfect concordance of the model prediction and the observed classes. If the prediction is in the opposite direction of the truth, a negative value will be obtained, but large negative values are rare in practice [@Kuhn2013].

We can plot accuracy (determined from repeated cross-validation) as a function of neighbourhood size.

```r
plot(knnFit)
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/cvAccuracyFunK-1.png" alt="Accuracy (repeated cross-validation) as a function of neighbourhood size." width="100%" />
<p class="caption">(\#fig:cvAccuracyFunK)Accuracy (repeated cross-validation) as a function of neighbourhood size.</p>
</div>

We can also plot other performance metrics, such as Cohen's Kappa, using the **metric** argument.

```r
plot(knnFit, metric="Kappa")
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/cvKappaFunK-1.png" alt="Cohen's Kappa (repeated cross-validation) as a function of neighbourhood size." width="100%" />
<p class="caption">(\#fig:cvKappaFunK)Cohen's Kappa (repeated cross-validation) as a function of neighbourhood size.</p>
</div>

Let us now evaluate how our classifier performs on the test set.

```r
test_pred <- predict(knnFit, xtest)
confusionMatrix(test_pred, ytest)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction   0   1
##          0 154  68
##          1  46 132
##                                          
##                Accuracy : 0.715          
##                  95% CI : (0.668, 0.7588)
##     No Information Rate : 0.5            
##     P-Value [Acc > NIR] : <2e-16         
##                                          
##                   Kappa : 0.43           
##  Mcnemar's Test P-Value : 0.0492         
##                                          
##             Sensitivity : 0.7700         
##             Specificity : 0.6600         
##          Pos Pred Value : 0.6937         
##          Neg Pred Value : 0.7416         
##              Prevalence : 0.5000         
##          Detection Rate : 0.3850         
##    Detection Prevalence : 0.5550         
##       Balanced Accuracy : 0.7150         
##                                          
##        'Positive' Class : 0              
## 
```

Scatterplots with decision boundaries can be plotted using the methods described earlier. First create a grid so we can predict across the full range of our variables V1 and V2:

```r
gridSize <- 150 
v1limits <- c(min(c(xtrain[,1],xtest[,1])),max(c(xtrain[,1],xtest[,1])))
tmpV1 <- seq(v1limits[1],v1limits[2],len=gridSize)
v2limits <- c(min(c(xtrain[,2],xtest[,2])),max(c(xtrain[,2],xtest[,2])))
tmpV2 <- seq(v2limits[1],v2limits[2],len=gridSize)
xgrid <- expand.grid(tmpV1,tmpV2)
names(xgrid) <- names(xtrain)
```

Predict values of all elements of grid.

```r
knn1grid <- predict(knnFit, xgrid)
V3 <- as.numeric(as.vector(knn1grid))
xgrid <- cbind(xgrid, V3)
```

Plot

```r
point_shapes <- c(15,17)
point_colours <- brewer.pal(3,"Dark2")
point_size = 2

ggplot(xgrid, aes(V1,V2)) +
  geom_point(col=point_colours[knn1grid], shape=16, size=0.3) +
  geom_point(data=xtrain, aes(V1,V2), col=point_colours[ytrain+1],
             shape=point_shapes[ytrain+1], size=point_size) +
  geom_contour(data=xgrid, aes(x=V1, y=V2, z=V3), breaks=0.5, col="grey30") +
  ggtitle("train") +
  theme_bw() +
  theme(plot.title = element_text(size=25, face="bold"), axis.text=element_text(size=15),
        axis.title=element_text(size=20,face="bold"))

ggplot(xgrid, aes(V1,V2)) +
  geom_point(col=point_colours[knn1grid], shape=16, size=0.3) +
  geom_point(data=xtest, aes(V1,V2), col=point_colours[ytest+1],
             shape=point_shapes[ytrain+1], size=point_size) +
  geom_contour(data=xgrid, aes(x=V1, y=V2, z=V3), breaks=0.5, col="grey30") +
  ggtitle("test") +
  theme_bw() +
  theme(plot.title = element_text(size=25, face="bold"), axis.text=element_text(size=15),
        axis.title=element_text(size=20,face="bold"))
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/simDataBinClassDecisionBoundaryK83-1.png" alt="Binary classification of the simulated training and test sets with _k_=83." width="50%" /><img src="07-nearest-neighbours_files/figure-html/simDataBinClassDecisionBoundaryK83-2.png" alt="Binary classification of the simulated training and test sets with _k_=83." width="50%" />
<p class="caption">(\#fig:simDataBinClassDecisionBoundaryK83)Binary classification of the simulated training and test sets with _k_=83.</p>
</div>

## Classification: cell segmentation {#knn-cell-segmentation}

The simulated data in our previous example were randomly sampled from a normal (Gaussian) distribution and so did not require pre-processing. In practice, data collected in real studies often require transformation and/or filtering. Furthermore, the simulated data contained only two predictors; in practice, you are likely to have many variables. For example, in a gene expression study you might have thousands of variables. When using _k_-nn for classification or regression, removing variables that are not associated with the outcome of interest may improve the predictive power of the model. The process of choosing the best predictors from the available variables is known as *feature selection*. For honest estimates of model performance, pre-processing and feature selection should be performed within the loops of the cross validation process.

### Cell segmentation data set 
Pre-processing and feature selection will be demonstrated using the cell segmentation data of (@Hill2007). High Content Screening (HCS) automates the collection and analysis of biological images of cultured cells. However, image segmentation algorithms are not perfect and sometimes do not reliably quantitate the morphology of cells. Hill et al. sought to differentiate between well- and poorly-segmented cells based on the morphometric data collected in HCS. If poorly-segmented cells can be automatically detected and eliminated, then the accuracy of studies using HCS will be improved. Hill et al. collected morphometric data on 2019 cells and asked human reviewers to classify the cells as well- or poorly-segmented.

<div class="figure" style="text-align: center">
<img src="images/Hill_2007_cell_segmentation.jpg" alt="Image segmentation in high content screening. Images **b** and **c** are examples of well-segmented cells; **d** and **e** show poor-segmentation. Source: Hill(2007) https://doi.org/10.1186/1471-2105-8-340" width="75%" />
<p class="caption">(\#fig:imageSegmentationHCS)Image segmentation in high content screening. Images **b** and **c** are examples of well-segmented cells; **d** and **e** show poor-segmentation. Source: Hill(2007) https://doi.org/10.1186/1471-2105-8-340</p>
</div>

This data set is one of several included in [caret](http://cran.r-project.org/web/packages/caret/index.html).

```r
data(segmentationData)
str(segmentationData)
```

```
## 'data.frame':	2019 obs. of  61 variables:
##  $ Cell                   : int  207827637 207932307 207932463 207932470 207932455 207827656 207827659 207827661 207932479 207932480 ...
##  $ Case                   : Factor w/ 2 levels "Test","Train": 1 2 2 2 1 1 1 1 1 1 ...
##  $ Class                  : Factor w/ 2 levels "PS","WS": 1 1 2 1 1 2 2 1 2 2 ...
##  $ AngleCh1               : num  143.25 133.75 106.65 69.15 2.89 ...
##  $ AreaCh1                : int  185 819 431 298 285 172 177 251 495 384 ...
##  $ AvgIntenCh1            : num  15.7 31.9 28 19.5 24.3 ...
##  $ AvgIntenCh2            : num  4.95 206.88 116.32 102.29 112.42 ...
##  $ AvgIntenCh3            : num  9.55 69.92 63.94 28.22 20.47 ...
##  $ AvgIntenCh4            : num  2.21 164.15 106.7 31.03 40.58 ...
##  $ ConvexHullAreaRatioCh1 : num  1.12 1.26 1.05 1.2 1.11 ...
##  $ ConvexHullPerimRatioCh1: num  0.92 0.797 0.935 0.866 0.957 ...
##  $ DiffIntenDensityCh1    : num  29.5 31.9 32.5 26.7 31.6 ...
##  $ DiffIntenDensityCh3    : num  13.8 43.1 36 22.9 21.7 ...
##  $ DiffIntenDensityCh4    : num  6.83 79.31 51.36 26.39 25.03 ...
##  $ EntropyIntenCh1        : num  4.97 6.09 5.88 5.42 5.66 ...
##  $ EntropyIntenCh3        : num  4.37 6.64 6.68 5.44 5.29 ...
##  $ EntropyIntenCh4        : num  2.72 7.88 7.14 5.78 5.24 ...
##  $ EqCircDiamCh1          : num  15.4 32.3 23.4 19.5 19.1 ...
##  $ EqEllipseLWRCh1        : num  3.06 1.56 1.38 3.39 2.74 ...
##  $ EqEllipseOblateVolCh1  : num  337 2233 802 725 608 ...
##  $ EqEllipseProlateVolCh1 : num  110 1433 583 214 222 ...
##  $ EqSphereAreaCh1        : num  742 3279 1727 1195 1140 ...
##  $ EqSphereVolCh1         : num  1901 17654 6751 3884 3621 ...
##  $ FiberAlign2Ch3         : num  1 1.49 1.3 1.22 1.49 ...
##  $ FiberAlign2Ch4         : num  1 1.35 1.52 1.73 1.38 ...
##  $ FiberLengthCh1         : num  27 64.3 21.1 43.1 34.7 ...
##  $ FiberWidthCh1          : num  7.41 13.17 21.14 7.4 8.48 ...
##  $ IntenCoocASMCh3        : num  0.01118 0.02805 0.00686 0.03096 0.02277 ...
##  $ IntenCoocASMCh4        : num  0.05045 0.01259 0.00614 0.01103 0.07969 ...
##  $ IntenCoocContrastCh3   : num  40.75 8.23 14.45 7.3 15.85 ...
##  $ IntenCoocContrastCh4   : num  13.9 6.98 16.7 13.39 3.54 ...
##  $ IntenCoocEntropyCh3    : num  7.2 6.82 7.58 6.31 6.78 ...
##  $ IntenCoocEntropyCh4    : num  5.25 7.1 7.67 7.2 5.5 ...
##  $ IntenCoocMaxCh3        : num  0.0774 0.1532 0.0284 0.1628 0.1274 ...
##  $ IntenCoocMaxCh4        : num  0.172 0.0739 0.0232 0.0775 0.2785 ...
##  $ KurtIntenCh1           : num  -0.6567 -0.2488 -0.2935 0.6259 0.0421 ...
##  $ KurtIntenCh3           : num  -0.608 -0.331 1.051 0.128 0.952 ...
##  $ KurtIntenCh4           : num  0.726 -0.265 0.151 -0.347 -0.195 ...
##  $ LengthCh1              : num  26.2 47.2 28.1 37.9 36 ...
##  $ NeighborAvgDistCh1     : num  370 174 158 206 205 ...
##  $ NeighborMinDistCh1     : num  99.1 30.1 34.9 33.1 27 ...
##  $ NeighborVarDistCh1     : num  128 81.4 90.4 116.9 111 ...
##  $ PerimCh1               : num  68.8 154.9 84.6 101.1 86.5 ...
##  $ ShapeBFRCh1            : num  0.665 0.54 0.724 0.589 0.6 ...
##  $ ShapeLWRCh1            : num  2.46 1.47 1.33 2.83 2.73 ...
##  $ ShapeP2ACh1            : num  1.88 2.26 1.27 2.55 2.02 ...
##  $ SkewIntenCh1           : num  0.455 0.399 0.472 0.882 0.517 ...
##  $ SkewIntenCh3           : num  0.46 0.62 0.971 1 1.177 ...
##  $ SkewIntenCh4           : num  1.233 0.527 0.325 0.604 0.926 ...
##  $ SpotFiberCountCh3      : int  1 4 2 4 1 1 0 2 1 1 ...
##  $ SpotFiberCountCh4      : num  5 12 7 8 8 5 5 8 12 8 ...
##  $ TotalIntenCh1          : int  2781 24964 11552 5545 6603 53779 43950 4401 7593 6512 ...
##  $ TotalIntenCh2          : num  701 160998 47511 28870 30306 ...
##  $ TotalIntenCh3          : int  1690 54675 26344 8042 5569 21234 20929 4136 6488 7503 ...
##  $ TotalIntenCh4          : int  392 128368 43959 8843 11037 57231 46187 373 24325 23162 ...
##  $ VarIntenCh1            : num  12.5 18.8 17.3 13.8 15.4 ...
##  $ VarIntenCh3            : num  7.61 56.72 37.67 30.01 20.5 ...
##  $ VarIntenCh4            : num  2.71 118.39 49.47 24.75 45.45 ...
##  $ WidthCh1               : num  10.6 32.2 21.2 13.4 13.2 ...
##  $ XCentroid              : int  42 215 371 487 283 191 180 373 236 303 ...
##  $ YCentroid              : int  14 347 252 295 159 127 138 181 467 468 ...
```
The first column of **segmentationData** is a unique identifier for each cell and the second column is a factor indicating how the observations were characterized into training and test sets in the original study; these two variables are irrelevant for the purposes of this demonstration and so can be discarded. 

The third column *Case* contains the class labels: *PS* (poorly-segmented) and *WS* (well-segmented). Columns 4-61 are the 58 morphological measurements available to be used as predictors. Let's put the class labels in a vector and the predictors in their own data.frame.

```r
segClass <- segmentationData$Class
segData <- segmentationData[,4:61]
```

### Data splitting
Before starting analysis we must partition the data into training and test sets, using the **createDataPartition** function in [caret](http://cran.r-project.org/web/packages/caret/index.html).

```r
set.seed(42)
trainIndex <- createDataPartition(y=segClass, times=1, p=0.5, list=F)
segDataTrain <- segData[trainIndex,]
segDataTest <- segData[-trainIndex,]
segClassTrain <- segClass[trainIndex]
segClassTest <- segClass[-trainIndex]
```

This results in balanced class distributions within the splits:

```r
summary(segClassTrain)
```

```
##  PS  WS 
## 650 360
```

```r
summary(segClassTest)
```

```
##  PS  WS 
## 650 359
```

_**N.B. The test set is set aside for now. It will be used only ONCE, to test the final model.**_

### Identification of data quality issues

Let's check our training data set for some undesirable characteristics which may impact model performance and should be addressed through pre-processing. 

#### Zero and near zero-variance predictors
The function **nearZeroVar** identifies predictors that have one unique value. It also diagnoses predictors having both of the following characteristics:

* very few unique values relative to the number of samples
* the ratio of the frequency of the most common value to the frequency of the 2nd most common value is large.

Such _zero and near zero-variance predictors_ have a deleterious impact on modelling and may lead to unstable fits.


```r
nzv <- nearZeroVar(segDataTrain, saveMetrics=T)
nzv
```

```
##                         freqRatio percentUnique zeroVar   nzv
## AngleCh1                 1.000000    100.000000   FALSE FALSE
## AreaCh1                  1.083333     37.326733   FALSE FALSE
## AvgIntenCh1              1.000000    100.000000   FALSE FALSE
## AvgIntenCh2              3.000000     99.801980   FALSE FALSE
## AvgIntenCh3              1.000000    100.000000   FALSE FALSE
## AvgIntenCh4              2.000000     99.900990   FALSE FALSE
## ConvexHullAreaRatioCh1   1.000000     98.910891   FALSE FALSE
## ConvexHullPerimRatioCh1  1.000000    100.000000   FALSE FALSE
## DiffIntenDensityCh1      1.000000    100.000000   FALSE FALSE
## DiffIntenDensityCh3      1.000000    100.000000   FALSE FALSE
## DiffIntenDensityCh4      1.000000    100.000000   FALSE FALSE
## EntropyIntenCh1          1.000000    100.000000   FALSE FALSE
## EntropyIntenCh3          1.000000    100.000000   FALSE FALSE
## EntropyIntenCh4          1.000000    100.000000   FALSE FALSE
## EqCircDiamCh1            1.083333     37.326733   FALSE FALSE
## EqEllipseLWRCh1          1.000000    100.000000   FALSE FALSE
## EqEllipseOblateVolCh1    1.000000    100.000000   FALSE FALSE
## EqEllipseProlateVolCh1   1.000000    100.000000   FALSE FALSE
## EqSphereAreaCh1          1.083333     37.326733   FALSE FALSE
## EqSphereVolCh1           1.083333     37.326733   FALSE FALSE
## FiberAlign2Ch3           1.304348     94.950495   FALSE FALSE
## FiberAlign2Ch4           7.285714     94.455446   FALSE FALSE
## FiberLengthCh1           1.000000     95.841584   FALSE FALSE
## FiberWidthCh1            1.000000     95.841584   FALSE FALSE
## IntenCoocASMCh3          1.000000    100.000000   FALSE FALSE
## IntenCoocASMCh4          1.000000    100.000000   FALSE FALSE
## IntenCoocContrastCh3     1.000000    100.000000   FALSE FALSE
## IntenCoocContrastCh4     1.000000    100.000000   FALSE FALSE
## IntenCoocEntropyCh3      1.000000    100.000000   FALSE FALSE
## IntenCoocEntropyCh4      1.000000    100.000000   FALSE FALSE
## IntenCoocMaxCh3          1.250000     94.158416   FALSE FALSE
## IntenCoocMaxCh4          1.250000     94.356436   FALSE FALSE
## KurtIntenCh1             1.000000    100.000000   FALSE FALSE
## KurtIntenCh3             1.000000    100.000000   FALSE FALSE
## KurtIntenCh4             1.000000    100.000000   FALSE FALSE
## LengthCh1                1.000000    100.000000   FALSE FALSE
## NeighborAvgDistCh1       1.000000    100.000000   FALSE FALSE
## NeighborMinDistCh1       1.166667     41.089109   FALSE FALSE
## NeighborVarDistCh1       1.000000    100.000000   FALSE FALSE
## PerimCh1                 1.000000     63.762376   FALSE FALSE
## ShapeBFRCh1              1.000000    100.000000   FALSE FALSE
## ShapeLWRCh1              1.000000    100.000000   FALSE FALSE
## ShapeP2ACh1              1.000000     99.801980   FALSE FALSE
## SkewIntenCh1             1.000000    100.000000   FALSE FALSE
## SkewIntenCh3             1.000000    100.000000   FALSE FALSE
## SkewIntenCh4             1.000000    100.000000   FALSE FALSE
## SpotFiberCountCh3        1.212000      1.287129   FALSE FALSE
## SpotFiberCountCh4        1.152778      3.267327   FALSE FALSE
## TotalIntenCh1            1.000000     98.712871   FALSE FALSE
## TotalIntenCh2            1.500000     99.009901   FALSE FALSE
## TotalIntenCh3            1.000000     99.108911   FALSE FALSE
## TotalIntenCh4            1.000000     99.603960   FALSE FALSE
## VarIntenCh1              1.000000    100.000000   FALSE FALSE
## VarIntenCh3              1.000000    100.000000   FALSE FALSE
## VarIntenCh4              1.000000    100.000000   FALSE FALSE
## WidthCh1                 1.000000    100.000000   FALSE FALSE
## XCentroid                1.111111     41.584158   FALSE FALSE
## YCentroid                1.000000     35.742574   FALSE FALSE
```

#### Scaling
The variables in this data set are on different scales, for example:

```r
summary(segDataTrain$IntenCoocASMCh4)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
## 0.004874 0.017253 0.049458 0.101586 0.121245 0.867845
```

```r
summary(segDataTrain$TotalIntenCh2)
```

```
##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
##       1   15846   49648   53143   72304  362465
```

In this situation it is important to centre and scale each predictor. A predictor variable is centered by subtracting the mean of the predictor from each value. To scale a predictor variable, each value is divided by its standard deviation. After centring and scaling the predictor variable has a mean of 0 and a standard deviation of 1. 


#### Skewness
Many of the predictors in the segmentation data set exhibit skewness, _i.e._ the distribution of their values is asymmetric, for example:

```r
qplot(segDataTrain$IntenCoocASMCh3, binwidth=0.1) + 
  xlab("IntenCoocASMCh3") +
  theme_bw()
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/segDataSkewness-1.png" alt="Example of a predictor from the segmentation data set showing skewness." width="75%" />
<p class="caption">(\#fig:segDataSkewness)Example of a predictor from the segmentation data set showing skewness.</p>
</div>

[caret](http://cran.r-project.org/web/packages/caret/index.html) provides various methods for transforming skewed variables to normality, including the Box-Cox [@BoxCox] and Yeo-Johnson [@YeoJohnson] transformations.

#### Correlated predictors

Many of the variables in the segmentation data set are highly correlated.

A correlogram provides a helpful visualization of the patterns of pairwise correlation within the data set.


```r
library(corrplot)
corMat <- cor(segDataTrain)
corrplot(corMat, order="hclust", tl.cex=0.4)
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/segDataCorrelogram-1.png" alt="Correlogram of the segmentation data set." width="75%" />
<p class="caption">(\#fig:segDataCorrelogram)Correlogram of the segmentation data set.</p>
</div>

The **preProcess** function in [caret](http://cran.r-project.org/web/packages/caret/index.html) has an option, **corr** to remove highly correlated variables. It considers the absolute values of pair-wise correlations. If two variables are highly correlated, **preProcess** looks at the mean absolute correlation of each variable and removes the variable with the largest mean absolute correlation. 

In the case of data-sets comprised of many highly correlated variables, an alternative to removing correlated predictors is the transformation of the entire data set to a lower dimensional space, using a technique such as principal component analysis (PCA). Methods for dimensionality reduction will be explored in chapter \@ref(dimensionality-reduction).

<!--

```r
highCorr <- findCorrelation(corMat, cutoff=0.75)
length(highCorr)
```

```
## [1] 31
```

```r
segDataTrain <- segDataTrain[,-highCorr]
```
-->


### Fit model without feature selection

<!-- original settings:
set.seed(42)
seeds <- vector(mode = "list", length = 101)
for(i in 1:100) seeds[[i]] <- sample.int(1000, 50)
seeds[[101]] <- sample.int(1000,1)
-->
Generate a list of seeds.

```r
set.seed(42)
seeds <- vector(mode = "list", length = 26)
for(i in 1:25) seeds[[i]] <- sample.int(1000, 50)
seeds[[26]] <- sample.int(1000,1)
```

Create a list of computational options for resampling. In the interest of speed for this demonstration, we will perform 5-fold cross-validation a total of 5 times. In practice we would use a larger number of folds and repetitions.

```r
train_ctrl <- trainControl(method="repeatedcv",
                   number = 5,
                   repeats = 5,
                   preProcOptions=list(cutoff=0.75),
                   seeds = seeds)
```
The ```cutoff``` refers to the correlation coefficient threshold.

Create a grid of values of _k_ for evaluation.

```r
tuneParam <- data.frame(k=seq(5,500,10))
```

To deal with the issues of scaling, skewness and highly correlated predictors identified earlier, we need to pre-process the data. We will use the Yeo-Johnson transformation to reduce skewness, because it can deal with the zero values present in some of the predictors. 
Perform cross validation to find best value of _k_.
```{r echo=T} <!--, message=F, warning=F-->
knnFit <- train(segDataTrain, segClassTrain, 
                method="knn",
                preProcess = c("YeoJohnson", "center", "scale", "corr"),
                tuneGrid=tuneParam,
                trControl=train_ctrl)

knnFit
```






```r
plot(knnFit)
```

<div class="figure" style="text-align: center">
<img src="07-nearest-neighbours_files/figure-html/cvAccuracySegDataHighCorRem-1.png" alt="Accuracy (repeated cross-validation) as a function of neighbourhood size for the segmentation training data with highly correlated predictors removed." width="100%" />
<p class="caption">(\#fig:cvAccuracySegDataHighCorRem)Accuracy (repeated cross-validation) as a function of neighbourhood size for the segmentation training data with highly correlated predictors removed.</p>
</div>


### Feature selection using filter

We will use the same **trainingControl** settings and **tuning grid** as before. 

```r
train_ctrl <- trainControl(method="repeatedcv",
                   number = 5,
                   repeats = 5
                   )
```

Let's define a filter using Caret's Selection By Filter (SBF) function:

```r
mySBF <- caretSBF
mySBF$summary <- twoClassSummary
```

We will use a simple t-test to eliminate the predictors that differ the least between classes. Since we are performing many hypothesis tests we will use Holm's method to control the family wise error rate.

```r
mySBF$score <- function(x, y) {
  out <- t.test(x ~ y)$p.value 
  out <- p.adjust(out, method="holm")
  out
}
```

Now to set a p-value threshold for our t-test filter.

```r
mySBF$filter <- function(score, x, y) { score <= 0.01 }
```

Let's run the cross-validation. The cross-validation will run in two nested loops. Feature selection will occur in the outer loop. Features selected at each iteration of the outer loop will be passed to the inner loop, where the optimum value of k will be found for that set of features.










































