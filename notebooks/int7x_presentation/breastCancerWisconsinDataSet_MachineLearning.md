# Table of Contents
+ [Model Estimation](#modelest) 
+ [Kth Nearest Neighbor](#knn) 
+ [Random Forest](#randforest)
+ [Neural Networks](#neurnetwork)
+ [ROC Curves](#roccurve)
+ [Conclusions](#conclusions)

# Model Estimation <a name='modelest'></a>
Now that we've gotten a *feel* for the data in [part 1](https://www.inertia7.com/projects/3), we are ready to begin predictive modeling. When going about predicitive modeling, especially when new data isn't readily available, creating a *training* and *test sets* will help in understanding your model's predictive power. 

We will use the *training set* to train our model, essentially learning from data it's seeing to later infer data it hasn't seen yet. We want to avoid using our entire data set on the training process, due to the process called *overfitting*. 

### Overfitting
We run the risk of *over-fitting* our data when we train the model on our entire dataset, especially true for this data set since we don't have any other data to see how well our data does. *Over-fitting* will cause our model to output strong predicitive power, but only for our training data usually performing poorly on new data.  

We avoid this through the use of the *training* and *test set*, where we measure the predicitive power on the *test set*. This will be a good indicator of our model's performance, but test sets also have their limitations. This is where *Cross Validation* will come into play, but more on this later. 

## Creating Training and Test Sets

We split the data set into our training and test sets which will be (pseudo) randomly selected having a *80-20%* splt. 

**NOTE**: What I mean when I say pseudo-random is that we would want everyone who replicates this project to get the same results especially if you're trying to learn from this project. So we use a random seed generator and set it equal to a number of our choosing, this will then make the results the same for anyone who uses this generator, awesome for reproducibility.  


```python
# Here we do a 80-20 split for our training and test set
train, test = train_test_split(breast_cancer, 
                               test_size = 0.20, 
                               random_state = 42)

# Create the training test omitting the diagnosis
training_set = train.ix[:, train.columns != 'diagnosis']
# Next we create the class set (Called target in Python Documentation)
# Note: This was confusing af to figure out 
# cus the documentation is low-key kind of shitty
class_set = train.ix[:, train.columns == 'diagnosis']

# Next we create the test set doing the same process as the training set
test_set = test.ix[:, test.columns != 'diagnosis']
test_class_set = test.ix[:, test.columns == 'diagnosis']
```

# Kth Nearest Neighbor <a name='knn'></a>

A popular algorithm within classification, *kth nearest neighbor* employs a majority votes methodology using *Euclidean Distance* (a.k.a straight-line distance between two points) based on the specificied *k*. 

So within context of my data set, I employ *k=3* so the algorithm looks for the 3 neighbors closest to the value its trying to classify (again closest measured using [Euclidean Distance](https://mathworld.wolfram.com/Distance.html)). This algorithm is known as a *lazy algorithm* and is considered one of the simpler algorithms in *Machine Learning*. 


One can often find that *K-nn* will perform better than more complicated methods, recall *Occam's Razor* when going about data analysis. 

**Important to Note**: The biggest drawback for *K-NN* is the [Curse of Dimensionality](https://en.wikipedia.org/wiki/Curse_of_dimensionality)  

### Set up the Algorithm
I will be employing `scikit-learn` algorithms for this process of my project, but in the future I would be interested in building the algorithms from "scratch". I'm not there yet with my learning though. 


```python
fit_knn = KNeighborsClassifier(n_neighbors=3)
```

### Train the Model

Next we train the model on our *training set* and the *class set*. The other models follow a similar pattern, so its worth noting here I had to call up the `'diagnosis'` column in my `class_set` otherwise I got an error code. 


```python
fit_knn.fit(training_set, class_set['diagnosis'])
```

### Terminal Output 

```
KNeighborsClassifier(algorithm='auto', leaf_size=30, metric='minkowski',
           metric_params=None, n_jobs=1, n_neighbors=3, p=2,
           weights='uniform')
```

## Determining Optimal k 
Determining the best *k* can be done mathematically by creating a `for-loop` that searches through multiple *k* values, we collect the `cross_val` scores which we then see which was the lowest *mean squared error* and receive the corresponding *k*. 
This section was heavily influenced by [Kevin Zakka's Blog Post](https://kevinzakka.github.io/2016/07/13/k-nearest-neighbor/) 

```
myKs = []
for i in range(0, 50):
    if (i % 2 != 0):
        myKs.append(i)

cross_vals = []
for k in myKs:
    knn = KNeighborsClassifier(n_neighbors=k)
    scores = cross_val_score(knn,
                             training_set, 
                             class_set['diagnosis'], 
                             cv = 10, 
                             scoring='accuracy')
    cross_vals.append(scores.mean())

MSE = [1 - x for x in cross_vals]
optimal_k = myKs[MSE.index(min(MSE))]
print("Optimal K is {0}".format(optimal_k))
```

### Terminal Output

```
Optimal K is 3
```

### Training Set Calculations
For this set, we calculate the accuracy based on the training set. I decided to include this to give context, but within *Machine Learning* processes we aren't concerned with the *training error rate* since it can suffer from high *bias* and *over-fitting*. 

As I stated previously we're more concerned in seeing how our model does against data it hasn't seen before. 


```python
# We predict the class for our training set
predictions_train = fit_knn.predict(training_set) 

# Here we create a matrix comparing the actual values vs. the predicted values
print(pd.crosstab(predictions_train, 
                  class_set['diagnosis'], 
                  rownames=['Predicted Values'], 
                  colnames=['Actual Values']))

# Measure the accuracy based on the trianing set
accuracy = fit_knn.score(training_set, class_set['diagnosis'])

print("Here is our accuracy for our training set: {0: .3f} "\
  .format(accuracy))
```
### Terminal Output 

```
Actual Values       0    1
Predicted Values          
0                 281   18
1                   5  151
Here is our accuracy for our training set:
  0.949
```


Here we get the training set error from the `.score()` function where test set calculations will follow a similar workflow. 
```python
train_error_rate = 1 - accuracy   
print("The train error rate for our model is: {0: .3f}"\
.format(train_error_rate))
```
### Terminal Output 

```
    The train error rate for our model is:
  0.051
```


## Cross Validation 
Cross validation is a powerful tool that is used for estimating the predicitive power of your model, which performs better than the conventional *training* and *test set*. What we are doing with *Cross Validation* is we are essentially creating multiple *training* and *test sets*. 

In our case we are creating 10 sets within our data set that calculates the estimations we have done already, but then averages the prediction error to give us a more accurate representation of our model's prediction power, since the model's performance can vary significantly when utilizing different *training* and *test sets*. 

**Suggested Reading**: For a more concise explanation of *Cross Validation* I recommend reading *An Introduction to Statistical Learnings with Applications in R*, specifically chapter 5.1!


## K-Fold Cross Validation 
Here we are employing *K-Fold Cross Validation*, more specifically 10 folds. So therefore we are creating 10 subsets of our data where we will be employing the *training* and *test set methodology* then averaging the accuracy for all folds to give us our estimatation. 

We could have done *Cross Validation* on our entire data set, but when it comes to tuning our parameters (more on this later) we still need to utilize a *test set*. This is more important for other models, since the only part we really have to optimize for *Kth Nearest Neighbors* is the *k*, but for the sake of consistency we will do *Cross Validation* on the *training set*.
```python
n = KFold(n_splits=10)

scores = cross_val_score(fit_knn, 
                         test_set, 
                         test_class_set['diagnosis'], 
                         cv = n)
print("Accuracy: {0: 0.2f} (+/- {1: 0.2f})"\
      .format(scores.mean(), scores.std() / 2))
```

### Terminal Output 

```
    Accuracy:  0.925 (+/-  0.025)
```


## Test Set Evaluations

Next we start doing calculations on our *test set*, this will be our form of measurement that will indicate whether our model truly will perform as well as it did for our *training set*. 

We could have simply over-fitted our model, but since the model hasn't seen the *test set* it will be an un-biased measurement of accuracy. 


```python
# First we predict the Dx for the test set and call it predictions
predictions = fit_knn.predict(test_set)

# Let's compare the predictions vs. the actual values
print(pd.crosstab(predictions, 
                  test_class_set['diagnosis'], 
                  rownames=['Predicted Values'], 
                  colnames=['Actual Values']))

# Let's get the accuracy of our test set
accuracy = fit_knn.score(test_set, test_class_set['diagnosis'])

# TEST ERROR RATE!!
print("Here is our accuracy for our test set: {0: .3f}"\
  .format(accuracy))
```

### Terminal Output 

```
Actual Values      0   1
Predicted Values        
0                 68   5
1                  3  38
```



```python
# Here we calculate the test error rate!
test_error_rate = 1 - accuracy
print("The test error rate for our model is: {0: .3f}"\
  .format(test_error_rate))
```
### Terminal Output 

```
    The test error rate for our model is:
  0.070
```


### Conclusion for K-NN

So as you can see our **K-NN** model did pretty well! The biggest set back was the number of samples it predicted to not have cancer when they actually had cancer. This will be a useful measurement as well since we are concerned with our models incorrectly predicting *begnin* when in actuality the sample is *malignant*. Since this would spell life-threatening mistakes if our models were to be applied in a real life situation. 

So we will keep this in mind when comparing models.

### Calculating for later use in ROC Curves

Here we calculate the *false positive rate* and *true positive rate* for our model. This will be used later when creating **ROC Curves** when comparing our models. I will go into more detail in a later section. 


```python
fpr, tpr, _ = roc_curve(predictions, test_class_set)
```

###  Calculations for Area under the Curve 
This function calculates the area under the curve which you will see the relevancy later in this project, but ideally we want it to be closest to 1 as possible. 


```python
auc_knn = auc(fpr, tpr)
```

# Random Forest <a name='randforest'></a>
Also known as *Random Decision Forest*, *Random Forest* is an entire forest of random uncorrelated decision trees. This is an extension of *Decision Trees* that will perform significantly better than a single tree because it corrects over-fitting. Here is a brief overview of the evolution of *CART* analysis:
+ Single Decision Tree (Single tree)
+ Bagging Trees (Multiple trees) [Model with all features, M, considered at splits, where M = all features]
+ Random Forest (Multiple trees) [Model with m features considered at splits, where m < M, typically m = sqrt(M)]

### Bagging Trees
*Decision Trees* tend to have *low bias and high variance*, a process known as *Bagging Trees* (*Bagging* = *Bootstrap Aggregating*) was an extension that does random sampling with replacement where after creating N trees it classifies on majority votes. This process reduces the variance while at the same time keeping the bias low. However, a downside to this process is if certain features are strong predictors then too many trees will employ these features causing correlation between the trees. 

Thus *Random Forest* aims to reduce this correlation by choosing only a subsample of the feature space at each split. Essentially aiming to make the trees more independent thereby reducing the variance.   

Generally, we aim to create 500 trees and use our m to be sqrt(M) rounded down. So since we have 30 features I will use 5 for my `max_features` parameter. I will be using the *Entropy Importance* metric. 

For this model we use an index known as the [Information Gain](https://en.wikipedia.org/wiki/Information_gain_in_decision_trees). 


### Variable Importance

A useful feature within what are known as *CART* (Classification And Regression Trees) is extracting which features where the most important when using *Entropy* within context of *information gain*. 

For this next process we will be grabbing the index of these features, then using a `for loop` to state which were the most important. 


```python
fit_RF = RandomForestClassifier(random_state = 42, 
    bootstrap=True,
    max_depth=4,
    criterion='entropy',
    n_estimators = 500)
```


```python
fit_RF.fit(training_set, class_set['diagnosis'])
```

### Terminal Output 

```
    RandomForestClassifier(bootstrap=True, class_weight=None, criterion='entropy',
            max_depth=4, max_features='auto', max_leaf_nodes=None,
            min_impurity_split=1e-07, min_samples_leaf=1,
            min_samples_split=2, min_weight_fraction_leaf=0.0,
            n_estimators=500, n_jobs=1, oob_score=False, random_state=42,
            verbose=0, warm_start=False)
```



## Variable Importance
We can gain information from our model through the concept of *variable importance*. This allows us to see which variables played an important role in the forest that was created. 

Using the `.feature_importances_` feature in **CART** models, we extract the index of our feature space then do a `for-loop` to receive the variables that had the most importance when using the *entropy* criterion. 
```python
importancesRF = fit_RF.feature_importances_
indicesRF = np.argsort(importancesRF)[::-1]
indicesRF
```

### Terminal Output 


    array([27, 23,  7, 22, 20,  6,  0,  2,  3, 26, 13, 21, 10,  1, 25, 28, 24,
            5, 12, 16,  4, 19, 29, 15, 18,  9, 17, 11,  8, 14])



This will give us the importance of the variables from largest to smallest, which we will then visualize. 
```python
# Print the feature ranking
print("Feature ranking:")

for f in range(30):
    i = f
    print("%d. The feature '%s' \
    has a Information Gain of %f" % (f + 1,
                namesInd[indicesRF[i]],
                importancesRF[indicesRF[f]]))
```

### Terminal Output 

```
Feature ranking:
1. The feature 'concave_points_worst' has a Information Gain of 0.143349
2. The feature 'area_worst' has a Information Gain of 0.123030
3. The feature 'perimeter_worst' has a Information Gain of 0.121727
4. The feature 'concave_points_mean' has a Information Gain of 0.110016
5. The feature 'radius_worst' has a Information Gain of 0.083120
6. The feature 'concavity_mean' has a Information Gain of 0.053719
7. The feature 'concavity_worst' has a Information Gain of 0.044348
8. The feature 'radius_mean' has a Information Gain of 0.044206
9. The feature 'perimeter_mean' has a Information Gain of 0.039387
10. The feature 'area_mean' has a Information Gain of 0.038195
11. The feature 'area_se' has a Information Gain of 0.030992
12. The feature 'texture_worst' has a Information Gain of 0.026350
13. The feature 'texture_mean' has a Information Gain of 0.018334
14. The feature 'radius_se' has a Information Gain of 0.015580
15. The feature 'compactness_worst' has a Information Gain of 0.012579
16. The feature 'symmetry_worst' has a Information Gain of 0.012515
17. The feature 'compactness_mean' has a Information Gain of 0.011184
18. The feature 'smoothness_worst' has a Information Gain of 0.010990
19. The feature 'perimeter_se' has a Information Gain of 0.009370
20. The feature 'concavity_se' has a Information Gain of 0.007509
21. The feature 'fractal_dimension_worst' has a Information Gain of 0.007260
22. The feature 'smoothness_mean' has a Information Gain of 0.005731
23. The feature 'fractal_dimension_se' has a Information Gain of 0.004470
24. The feature 'symmetry_se' has a Information Gain of 0.004147
25. The feature 'concave_points_se' has a Information Gain of 0.004140
26. The feature 'fractal_dimension_mean' has a Information Gain of 0.003991
27. The feature 'compactness_se' has a Information Gain of 0.003982
28. The feature 'texture_se' has a Information Gain of 0.003522
29. The feature 'smoothness_se' has a Information Gain of 0.003234
30. The feature 'symmetry_mean' has a Information Gain of 0.003022
```

The same process can be done for any **CART** when looking at the *Variable Importance*.

### Feature Importance Visual

Here I use the `sorted` function to sort the *Information Gain* criterion from least to greatest which was a work around in order to create a horizontal barplot, as well as creating an index using the `arange` function in `numpy`


```python
indRf = sorted(importancesRF) # Sort by Decreasing order
index = np.arange(30)
index
```
### Terminal Output 



    array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14, 15, 16,
           17, 18, 19, 20, 21, 22, 23, 24, 25, 26, 27, 28, 29])



This next step we're creating an index that's backwards since we're creating a barplot that is horizontal we have to state the correct order. If we don't do this step then the names will be in reverse order (we don't want that).
```python
feature_space = []
for i in range(29, -1, -1):
    feature_space.append(namesInd[indicesRF[i]])
```

Now let's plot it. 


```python
f, ax = plt.subplots(figsize=(11, 11))

ax.set_axis_bgcolor('#fafafa')
plt.title('Feature importances for Random Forest Model')
plt.barh(index, indRf,
        align="center", 
        color = '#875FDB')

plt.yticks(index, feature_space)
plt.ylim(-1, 30)
plt.xlim(0, 0.15)
plt.xlabel('Information Gain')
plt.ylabel('Feature')

plt.show()
```
<img src='https://raw.githubusercontent.com/raviolli77/machineLearning_breastCancer_Python/master/images/varImprt.png'>

## Cross Validation 
Again here we employ a 10 fold *cross validation* method to get another accuracy estimation for our models. 

```python
n = KFold(n_splits=10)
scores = cross_val_score(fit_RF, 
                         test_set, 
                         test_class_set['diagnosis'], 
                         cv = n)

print("Accuracy: {0: 0.2f} (+/- {1: 0.2f})"\
      .format(scores.mean(), scores.std() / 2))
```
### Terminal Output 

```
Accuracy:  0.963 (+/-  0.013)
```


## Test Set Evaluations
These processes are similar to the previous *test set evaluation* so explanations won't be necessary. 

```python
predictions_RF = fit_RF.predict(test_set)
```


```python
print(pd.crosstab(predictions_RF, 
                  test_class_set['diagnosis'], 
                  rownames=['Predicted Values'], 
                  colnames=['Actual Values']))
```
### Terminal Output 

```
Actual Values      0   1
Predicted Values        
0                 70   3
1                  1  40
```



```python
accuracy_RF = fit_RF.score(test_set, test_class_set['diagnosis'])

print("Here is our mean accuracy on the test set:\n {0:.3f}"\
      .format(accuracy_RF))
```
### Terminal Output 

    Here is our mean accuracy on the test set:
     0.965


And now our test error rate.

```python
# Here we calculate the test error rate!
test_error_rate_RF = 1 - accuracy_RF
print("The test error rate for our model is:\n {0: .3f}"\
      .format(test_error_rate_RF))
```
### Terminal Output 

    The test error rate for our model is:
      0.035


### Calculating for later use in ROC Curves


```python
fpr2, tpr2, _ = roc_curve(predictions_RF, 
                          test_class_set)
```

### Calculations for  Area under Curve


```python
auc_rf = auc(fpr2, tpr2)
```

## Conclusions for Random Forest
Our *Random Forest* performed pretty well, I have a personal preference to tree type models because they are the most interprettable and give insight to data that some other models don't (for instance **K-NN**). We were able to see which variables were important when the *random forest* was created and thus we can expand on our data/model if we choose too, through the variable importance of *random forest*. 
That can be an exercise for later on to see if choosing a subset of the data set will help in prediction power. For now I am content with using the entire data set for instructive purposes. 

Important to note, is that the *random forest* performed better in terms of *false negatives*, only slightly though. 

# Neural Network <a name='neurnetwork'></a>
Our last model is a harder concept to understand. *Neural Networks* are considered *black box models*, meaning that we don't know what goes on under the hood, we feed the model our data and it outputs the predictions without giving us insight into how our model works. 

*Neural Network* architecture is very complex and I will do no justice explaining it in a few sentences, but I will try to give a brief overview to help with giving context. 
So there are three main components that make up an *artificial neural network*:
+ **Input Layer**: This is where we feed data into the model, we have a *neuron* for every variable in our feature space (in our case we have 30 input layers)
+ **Hidden Layer(s)**: This is where the magic happens, we have several hidden layers with activation functions (typically logisitic activation functions) that create weights and corrects itself through [Backpropagation](https://en.wikipedia.org/wiki/Backpropagation) and [Gradient descent](https://en.wikipedia.org/wiki/Gradient_descent) for further optimization
    + **Backpropagation**: The inputs are fed into the *neural network*, so they go through the hidden layers until they reach the output layer (forward propogation), which then outputs a prediction, the model then compares the prediction to the actual values utilizing a [loss function](https://en.wikipedia.org/wiki/Loss_function) to go backwards to optimize the hidden layers (through weights) using *gradient descent*
    + **Gradient Descent**: 
+ **Output Layer**: This section represents the output of the model so in our data set our output layer is binary

Here you can see a visual representation of a simple *neural network*:

<img src='http://cs231n.github.io/assets/nn1/neural_net.jpeg'>

## Data Leakage

A hidden pitfall in machine learning is a process known, as *Data Leakage*, data leakage is the process of letting information into your algorithm that it would otherwise not know. 

A concrete example using this data set, would be if I pre-maturely normalized my data set(which I did), but the leakage comes if I were to create my *training* and *test sets* through using the `normalize_data_frame` function I created. 

This is because the *training set* would see information about the test set that it should not have, thereby producing data leakage. Luckily for us, `sklearn.preprocessing` has classes that avoid this issue, but it is important to state.  

Important to note is that *neural networks* require a lot of pre-processing in order for them to be effective models. The standard is to standardize your dataframe, or else the model will perform noticeably poor. 

So here we are creating a new *training* and *test set* to input into our *neural network*. 

```python
# Scaling dataframe
scaler = MinMaxScaler().fit(training_set)

training_set_scaled = scaler.fit_transform(training_set)

test_set_scaled =  scaler.transform(test_set)

```


Here we create our *neural network* utilizing 5 hidden layers, the logisitic activiation function, and 
We won't be using *Gradient Descent* in this model instead we will be utilizing `lbfgs` which is an optimizer from quasi-Newton methods family, which according to the documentation:

*"For small datasets, however, ‘lbfgs’ can converge faster and perform better."* [Source](http://scikit-learn.org/stable/modules/generated/sklearn.neural_network.MLPClassifier.html)

Since our data set is relatively small I decided to go with `lbfgs`.

```python
fit_NN = MLPClassifier(solver='lbfgs', 
                       hidden_layer_sizes=(5, ), 
                       activation='logistic',
                       random_state=7)
```
## Train the Model 
Similar to the past two other models we train our model utilizing the *training set*. 

```python
fit_NN.fit(training_set_norm, 
           class_set_norm['diagnosis'])
```
### Terminal Output 


```
MLPClassifier(activation='tanh', alpha=0.0001, batch_size='auto', beta_1=0.9,
       beta_2=0.999, early_stopping=False, epsilon=1e-08,
       hidden_layer_sizes=(12,), learning_rate='constant',
       learning_rate_init=0.05, max_iter=200, momentum=0.9,
       nesterovs_momentum=True, power_t=0.5, random_state=42, shuffle=True,
       solver='lbfgs', tol=0.0001, validation_fraction=0.1, verbose=False,
       warm_start=False)
```


## Cross Validation 
Likewise we are utilizing 10 fold *cross validation* for this model. 

```python
n = KFold(n_splits=10)
scores = cross_val_score(fit_NN, 
                         test_set_norm, 
                         test_class_set_norm['diagnosis'], 
                         cv = n)

print("Accuracy: {0: 0.2f} (+/- {1: 0.2f})"\
      .format(scores.mean(), scores.std() / 2))
```
### Terminal Output 

    Accuracy:  0.976 (+/-  0.013)


Our model performed significantly worse then the other two models. Let's calculate the *test error rate* to compare its performance. 
## Test Set Calculations
Let's do the same test set calculations and compare all our models. 
```
predictions_NN = fit_NN.predict(test_set_norm)

print(pd.crosstab(predictions_NN, test_class_set_norm['diagnosis'], 
                  rownames=['Predicted Values'], 
                  colnames=['Actual Values']))```


### Terminal Output

```
Actual Values      0   1
Predicted Values        
0                 70   2
1                  1  41

```
The *Neural Network* model performed better in terms of *false negative*!
 
```python
accuracy_NN = fit_NN.score(test_set_norm, test_class_set_norm['diagnosis'])

print("Here is our mean accuracy on the test set:\n{0: .2f} "\
  .format(accuracy_NN))
```
### Terminal Output 

```
Here is our mean accuracy on the test set:
  0.974
```


```python
# Here we calculate the test error rate!
test_error_rate_NN = 1 - accuracy_NN
print("The test error rate for our model is:\n {0: .3f}"\
      .format(test_error_rate_NN))
```
### Terminal Output 

```
The test error rate for our model is:
  0.026
```

Next we do more *ROC* curve calculations. 

```python
fpr3, tpr3, _ = roc_curve(predictions_NN, test_class_set)
```


```python
auc_nn = auc(fpr3, tpr3)
```
## Conclusions for Neural Networks 
In terms of *cross validation*, our *neural network* peformed worse then the other two models. But this was expected since *neural networks* require a lot of *training data* to perform at its best, and unfortunately for us we only have the data that was provided in the *UCI repository*. 

# ROC Curves <a name='roccurve'></a>

*Receiver Operating Characteristc* Curve calculations we did using the function `roc_curve` were calculating the **False Positive Rates** and **True Positive Rates** for each model. We will now graph these calculations, and being located the top left corner of the plot indicates a really ideal model, i.e. a **False Positive Rate** of 0 and **True Positive Rate** of 1, so we plot the *ROC Curves* for all our models in the same axis and see how they compare!



We also calculated the **Area under the Curve**, so our curve in this case are the *ROC Curves*, we then place these calculations in the legend with their respective models. 


```python
f, ax = plt.subplots(figsize=(10, 10))

plt.plot(fpr, tpr, label='K-NN ROC Curve (area = {0: .3f})'.format(auc_knn), 
         color = 'deeppink', 
         linewidth=1)

plt.plot(fpr2, tpr2,label='Random Forest ROC Curve (area = {0: .3f})'\
    .format(auc_rf), 
         color = 'red', 
         linestyle=':', 
         linewidth=2)

plt.plot(fpr3, tpr3,label='Neural Networks ROC Curve (area = {0: .3f})'\
    .format(auc_nn), 
         color = 'purple', 
         linestyle=':', 
         linewidth=3)
    
ax.set_axis_bgcolor('#fafafa')
plt.plot([0, 1], [0, 1], 'k--', lw=2)
plt.plot([0, 0], [1, 0], 'k--', lw=2, color = 'black')
plt.plot([1, 0], [1, 1], 'k--', lw=2, color = 'black')
plt.xlim([-0.01, 1.0])
plt.ylim([0.0, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve Comparison For All Models')
plt.legend(loc="lower right")
    
plt.show()
```


<img src="https://raw.githubusercontent.com/raviolli77/machineLearning_breastCancer_Python/master/images/roc.png">

Let's zoom in to get a better picture!

### ROC Curve Plot Zoomed in


```python
f, ax = plt.subplots(figsize=(10, 10))
plt.plot(fpr, tpr, label='K-NN ROC Curve  (area = {0: .3f})'\
         .format(auc_knn), 
         color = 'deeppink', 
         linewidth=1)

plt.plot(fpr2, tpr2,label='Random Forest ROC Curve  (area = {0: .3f})'\
    .format(auc_rf), 
         color = 'red', 
         linestyle=':', 
         linewidth=3)

plt.plot(fpr3, tpr3,label='Neural Networks ROC Curve  (area = {0: .3f})'\
    .format(auc_nn), 
         color = 'purple', 
         linestyle=':', 
         linewidth=3)
    
ax.set_axis_bgcolor('#fafafa')
plt.plot([0, 1], [0, 1], 'k--', lw=2) # Add Diagonal line
    
plt.plot([0, 0], [1, 0], 'k--', lw=2, color = 'black')
plt.plot([1, 0], [1, 1], 'k--', lw=2, color = 'black')
plt.xlim([-0.001, 0.2])
plt.ylim([0.7, 1.05])
plt.xlabel('False Positive Rate')
plt.ylabel('True Positive Rate')
plt.title('ROC Curve Comparison For All Models (Zoomed)')
plt.legend(loc="lower right")
    
plt.show()
```


<img src="https://raw.githubusercontent.com/raviolli77/machineLearning_breastCancer_Python/master/images/roc_zoomed.png">

Visually examining the plot, *Random Forest* and *Neural Networks* are noticeable more elevated than *Kth Nearest Neighbor* which is indicative of a good prediction tool, using this form of diagnositcs. 

# Conclusions <a name='conclusions'></a>

When choosing models it isn't just about having the best accuracy, we are also concerned with insight because these models can help tell us which dependent variables are indicators thus helping researchers in the respective field to focus on the variables that have the most statistically significant influence in predictive modeling within the respective domain. Although this data set was engineered to facilitate machine learning processing employing the methodologies, I would say using both **Random Forest** and **Neural Networks** would help in predicting and getting insight into the data. 


### Diagnostics for Data Set

| Model/Algorithm      | Test Error Rate | False Negative for Test Set | Area under the Curve for ROC | Cross Validation Score        |
|----------------------|-----------------|-----------------------------|------------------------------|-------------------------------|
| Kth Nearest Neighbor | 0.07            | 5                           | 0.929                        | Accuracy:  0.925 (+/-  0.025) |
| Random Forest        | 0.035           | 3                           | 0.967                        | Accuracy:  0.963 (+/-  0.013) |
| Neural Networks      | 0.026           | 1                           | 0.974                        | Accuracy:  0.976 (+/-  0.013) |


## Classification Report

The classification report is available through `sklearn.metrics`, this report gives many important classification metrics including:
+ **Precision**: also the positive predictive value, is the number of correct predictions divided by the number of correct predictions plus false positives, so `tp / (tp + fp)`
+ **Recall**: also known as the sensitivity, is the number of correct predictions divided by the total number of instances so `tp / (tp + fn)` where `fn` is the number of false negatives
+ **f1-score**: this is defined as the *weighted harmonic mean* of both the precision and recall, where the f1-score at 1 is the best value and worst value at 0, as defined by the [documentation](http://scikit-learn.org/stable/modules/generated/sklearn.metrics.precision_recall_fscore_support.html#sklearn.metrics.precision_recall_fscore_support)
+ **support**: number of instances that are the correct target values

These were done using the function in the `helper_functions` script:

```
def print_class_report(predictions, alg_name):
    '''
    Purpose
    ----------
    Function helps automate the report generated by the
    sklearn package
    
    Parameters
    ----------
    * predictions:  The predictions made by the algorithm used
    * alg_name:     String containing the name of the algorithm used
    '''
    target_names = ['Benign', 'Malignant']
    print('Classification Report for {0}:'.format(alg_name))
    print(classification_report(predictions, 
            test_class_set['diagnosis'], 
            target_names = target_names))
```

Then we input the predictions as such into the function:

```
print_class_report(predictions_knn, 'Kth Nearest Neighbor') 
print_class_report(predictions_rf, 'Random Forest') 
print_class_report(predictions_nn, 'Neural Networks')
```

## Terminal Output

#### Classification Report for Kth Nearest Neighbor:

|              | precision  |   recall |  f1-score |   support | 
|--------------|------------|----------|-----------|-----------|
|      Benign  |      0.96  |     0.93 |      0.94 |        73 | 
|   Malignant  |      0.88  |     0.93 |      0.90 |        41 | 
| avg / total  |      0.93  |     0.93 |      0.93 |       114 | 

#### Classification Report for Random Forest:

|             | precision  |  recall | f1-score  | support | 
|--------------|------------|----------|-----------|-----------|
|      Benign |      0.99  |    0.96 |     0.97  |      73 | 
|   Malignant |      0.93  |    0.98 |     0.95  |      41 | 
| avg / total |      0.97  |    0.96 |     0.97  |     114 | 

#### Classification Report for Neural Networks:

|             | precision |    recall |  f1-score |   support |
|-------------|-----------|-----------|-----------|-----------|
|      Benign |      0.99 |      0.97 |      0.98 |        72 |
|   Malignant |      0.95 |      0.98 |      0.96 |        42 |
| avg / total |      0.97 |      0.97 |      0.97 |       114 |

Once I employed all these methods, we can that **Neural Networks** performed the best in terms of most diagnostics, see more detail below. **Random Forest** helped with identifying important variables and I would say employing both models would help give important insight. 


# Sources Cited

+ W. Street, Nick. UCI Machine Learning Repository [http://archive.ics.uci.edu/ml]. Irvine, CA: University of California, School of Information and Computer Science.
+ Waidyawansa, Buddhini. Kaggle [https://www.kaggle.com/]. *Using the Wisconsin breast cancer diagnostic data set for predictive analysis*. [Kernel Source](https://www.kaggle.com/buddhiniw/breast-cancer-prediction)
+ Zakka, Kevin. Kevin Zakka's Blog [https://kevinzakka.github.io/]. *A Complete Guide to K-Nearest-Neighbors with Applications in Python and R*. [Blog Source](https://kevinzakka.github.io/2016/07/13/k-nearest-neighbor/)