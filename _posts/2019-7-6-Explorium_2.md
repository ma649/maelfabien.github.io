---
published: true
title: Interpretability and Explainability
collection: explorium
layout: single
author_profile: false
read_time: true
categories: [machinelearning]
excerpt : "Better ML"
header :
    overlay_image: "https://maelfabien.github.io/assets/images/wolf.jpg"
    teaser : "https://maelfabien.github.io/assets/images/wolf.jpg"
comments : true
toc: true
toc_sticky: true
search: false
sidebar:
    nav: sidebar-sample
---

<script type="text/javascript" async
src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-MML-AM_CHTML">
</script>

Machine Learning interpretability and explainability are becoming essential in solutions we build nowadays. In fields such as health care or banking, there exists legal restrictions that interpretability and explainability could help overcome. In solutions that support a human decision, it is essential to build a trust relationship and explain the outcome of an algorithm. The whole idea behind interpretable and explainable ML is to avoid the black box effect.

Christop Molnar has recently publshed an excellent book on this topic : [Interpretable Machine Learning](https://christophm.github.io/interpretable-ml-book/).

Fist of all, let's define the difference between machine learning explainability and interpretability :
- Interpretability is linked to the model. A model is said to be interpretable if its parameters are linked in a clear way to the impact of a feature on the outcome. In some sense, it is the extent to which we are able to predict what is going to happen, given a change in input. Among interpretable models, one can for example mention :
    - Linear regression
    - Logistic regression
    - Decision trees
- Explainability can be applied to any model, even models that are not interpretable. Explainability is the extent to which we can interpret the outcome and the internal mechanics of an algorithm. 

# I. Interpretable models

## 1. Linear Regression

Linear regression is one of the the most basic models : $$ Y_i =  f(X_i) + {\epsilon} $$. This simple equation states the following :
- suppose we have $$ n $$ observations of a dataset and we pick on $$ i^{th} $$
- $$ Y_i $$ is the output of this observation, called the target
- $$ X_i $$ is called a feature and is an independent variable we observe
- $$ f $$ is the real model that states the link between the features and the output
- $$ {\epsilon} $$ is the noise of the model. The data we observe usually do not stand on a straight line, because there are variations of the measure in real life. 

For example, we can plot a linear regression between the GDP and the housing price. The model is the following : $$ Y_i = {\beta}_0 + {\beta}_1{X}_{i} + {\epsilon}_i $$, where $$ X $$ is the GDP and $$ Y $$ the housing price.
- $$ {\beta}_0 $$ is called the intercept, it is a constant term
- $$ {\beta}_1 $$ is the coefficient associated with $$ X_i $$ . It describes the weight of $$ X_i $$ on the final output.
- $$ {\epsilon}_i $$ is called the residual. It is a white noise term that explains the variability in real life datas. 

![image](https://maelfabien.github.io/assets/images/Graph2.jpg)

If we have a dataframe (`df`) with both the GDP and the housing price, we can build a simple model the following way :

```python
import sklearn.linear_model as lm

X = df[['gdp']]
y = df['house']

skl_linmod = lm.LinearRegression(fit_intercept = True).fit(X,y)
beta1_sk = skl_linmod.coef_[0]
beta0_sk = skl_linmod.intercept_
```

If we have more features observed (other than the GDP), we build a more complex model :  $$ \hat{Y}_i = \hat{\beta}_0 + \hat{\beta}_1{X}_{1i} + \hat{\beta}_2{X}_{2i} $$. The more general framework can be described by : $$ Y =  X {\beta}+ {\epsilon} $$ 

### Interpretability of Linear Regression

- The coefficients of a linear regression are directly interpretable. At stated above, each coefficient describes the effect on the output of a change of 1 unit of a given input. 
- The importance of a feature can be seen as the absolute value of the t-statistic value. The more variance the estimated weight has, the less important the feature is. The higher the estimated coefficient, the more important the feature is.

$$ t_{\hat{\beta_j}} = \frac{\hat{\beta_j}}{SE(\hat{\beta_j})} $$
- The variance explained by the model can be explained by the $$ R^2 $$ coefficient
- We can use confidence intervals and tests for coefficient values
- We are guaranteed to find the best coefficients by OLS properties

### Limitations of Linear Regression

- Linear regression is a basic model. It is rare to observe linear relationships in the data, and the linear regression is rarely performing well.
- Moreover, when it comes to classification tasks, the linear regression is risky to apply, since an hyperplane is not a good way to send the output between 0 and 1. We usually prefer to apply the Logistic Regression.

![image](https://maelfabien.github.io/assets/images/log_1.png)

## 2. Logistic Regression

The logistic regression using the logistic function to send the output between 0 and 1 for binary classification purposes. The function is defined as :

$$ Sig(z) = \frac {1} {1 + e^{-z}} $$

![image](https://maelfabien.github.io/assets/images//log_2.png)

In the logistic regression model, instead of a linear relation between the input and the output, the relation is the following :

$$ P(Y=1) = \frac {1} {1 + exp^{-(\beta_0 + \beta_1 X_1 + ... + \beta_p X_p)}} $$

How can we interpret the partial effect of $$ X_1 $$ on $$ Y $$ for example ? Well, the weights in the logistic regression **cannot** be interpreted as for linear regression. We need to use the logit transform :

$$ \log( \frac {P(y=1)} {1-P(y=1)} ) = \log ( \frac {P(y=1)} {P(y=0)} ) $$ 

$$ = odds = \beta_0 + \beta_1 X_1 + ... + \beta_k X_k $$

We define the this ratio as the "odds". Therefore, to estimate the impact of $$ X_j $$ increasing by 1 unit, we can compute it this way :

$$ \frac {odds_{X_{j+1}}} {odds} = \frac {exp^{\beta_0 + \beta_1 X_1 + ... + \beta_j (X_j + 1) + ... + \beta_k X_k}} {exp^{\beta_0 + \beta_1 X_1 + ... + \beta_j X_j + ... + \beta_k X_k}} $$

$$ = exp^{\beta_j (X_j + 1) - \beta_j X_j} = exp^{\beta_j} $$

A change in $$ X_j $$ by one unit increases the log odds ratio by the value of the corresponding weight : $$ exp^{\beta_j} $$.

The implementation is straight forward in Python using scikit-learn. We can for example use breast cancer data :

```python
# Imports
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score
from sklearn.metrics import f1_score

data = load_breast_cancer()
```

We then split the data into train and test :

```python
X_train, X_test, y_train, y_test = train_test_split(data['data'], data['target'], test_size = 0.25)
```

By default, L2-Regularization is implemented. Using L1-Regularization, we achieve :

```
lr = LogisticRegression(penalty='l1')
lr.fit(X_train, y_train)

y_pred = lr.predict(X_test)
print(accuracy_score(y_pred, y_test))
print(f1_score(y_pred, y_test))
```

```
0.958041958041958
0.9655172413793104
```

To print the coefficients, simply use :

```python
lr.coef_
```

### Interpretability of Logistic Regression

With the logistic regression, we can still apply test hypothesis, build confidence intervals and compute the explained variance. Many of the advantages of the linear regression remain.

### Limitations of Logistic Regression

Just like linear regression, the model remains quite limited in terms of performance, although a good regularization can offer decent performance. The coefficients are not as easily interpretable as for the linear regression. There is a tradeoff to make when choosing these kind of models, and they are often used in customer classification for car rental companies or in banking industry for example.

## 3. Decision Trees

Linear regression and logistic regression fail when features interact with each other.

![image](https://maelfabien.github.io/assets/images/dt.png)

The Classification And Regression Trees (CART) algorithm is the most simple and popular tree algorithm.

To build the tree, we choose each time the feature that splits our data the best way possible. How do we measure the qualitiy of a split ? Let $ p_{i} $ be the fraction of items labeled with class i in the set :
- Cross-entropy : $ H(p,q) = -\sum_{x \in {\mathcal{X}}} p(x) \log q(x) $ 
- Gini impurity : $ I_G = 1 - \sum_{i = 1...J} {p_i}^2 $
- classification error

Note that in order to grow a decision tree for numeric data, we usually order the data by value of each feature, compute the average between every successive pair of values, and compute the split quality measure (e.g Gini) using this average.

We stop the development of the tree when splitting a node does not lower the impurity.

The relationship between the inputs and the output is given by :

$$ \hat{y}=\hat{f}(x)=\sum_{m=1}^Mc_m{}I\{x\in{}R_m\} $$ where $$ R_m $$ denotes the subset $$ m $$. If an observation to predict falls into the subset $$ R_j $$, the predicted value is $$ c_j $$, the average value of all training instances that fell in this subset.

### Interpretability of CART algorithm

By growing the depth of the tree, we add "AND" conditions. For a new instance, the feature 1 is larger than `a` **and** the feature 3 is smaller than `b` **and** the feature 2 equals `c`.

CART algorithm offers a nice way to compute the importance of each feature in the model. We measure the importance of a Gini index by the extent to which the chosen citeria has been decreased when creating a new node on the given feature.

The tree offers a natural interpretability, and can be represented visually.

### Limitations of CART algorithm

CART algorithms fails to represent linear relationships between the input and the output. It easily overfits and gets quite deep if we don't crontrol the model. For this reason, tree based ensemble models such as Random Forest have been developped.

## 4. Other models

There are other models that are by construction interpretable :
- K-Nearest Neighbors (KNN)
- Generalized Linear Models (GLMs)
- Stochastic processses such as Poisson processes if you want to model arrival rates or goals in a football match
- ...

# II. Model explainability

If we don't have to use interpretable models and need higher performance, we tend to use black box models such as XGBoost for example. It might however be needed, for various reasons, to provide an explaination of the outcome and the inner mechanics of the model. In such case, using model explainability techniques is the right choice. 

Explainability is useful for :
- establishing trust in an outcome
- debugging
- legal restrictions
- ... 

The main questions model explainability answers are :
- what are the most important features ?
- how do you decompose a single prediction and the contribution of each feature ?
- how do you decompose the model outcome and the contribution of each feature ?

We will explore several techniques of model explainability :
- individual conditional expectation
- permutation importance
- partial dependence plots
- shapley values

## 1. Individual Conditional Expectation (ICE)

How does the prediction change when 1 feature changes ? Individual Conditional Expectation, as its name suggests, is a plot that shows how a change in an individual feature changes the outcome of each individual prediction (one line per prediction). 





## 2. Permutation importance

What features have the biggest impact on predictions? There are many ways to compute feature importance. We will focus on permutation importance, which is :
- fast to compute
- widely used
- consistent with the properties needed

### How does it work?

Permutation importance is computed after a model has been fitted. It answers the following question :
If with randomly shuffle a single column of the validation data, leaving the target and all other columns in place, how would that affect the accuracy?

For example, say we want to predict the height of a person at age 20 based on a set of features, including some less relevant ones (the number of socks owned at age 10):

![image](https://maelfabien.github.io/assets/images/perm.jpg)

Randomly re-ordering a single column should decrease the accuracy. Depending on how relevant the feature is, it will more or less impact the accuracy. From the impact on the accuracy, we can determine the importance of a feature.

### Example

In this example, we will try to predict the "Man of the Game" of a football match based on a set of features of a player in a match.

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier

data = pd.read_csv('../input/fifa-2018-match-statistics/FIFA 2018 Statistics.csv')

y = (data['Man of the Match'] == "Yes")  # Convert from string "Yes"/"No" to binary
feature_names = [i for i in data.columns if data[i].dtype in [np.int64]]
X = data[feature_names]

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=1)
my_model = RandomForestClassifier(random_state=0).fit(X_train, y_train)
```

We can then compute the Permutation Importance with [Eli5 library](https://eli5.readthedocs.io/en/latest/). Eli5 is a Python library which allows to visualize and debug various Machine Learning models using unified API. It has built-in support for several ML frameworks and provides a way to explain black-box models.

```python
import eli5
from eli5.sklearn import PermutationImportance

perm = PermutationImportance(my_model, random_state=1).fit(X_test, y_test)
eli5.show_weights(perm, feature_names = val_X.columns.tolist())
```

![image](https://maelfabien.github.io/assets/images/perm2.jpg)

In our example, the most important feature was Goals scored. The first number in each row shows how much model performance decreased with a random shuffling (in this case, using "accuracy" as the performance metric). We measure the randomness by repeating the process with multiple shuffles.

Negative value for importance occurs when the feature is not important at all.

## 3. Partial dependence plots

While feature importance shows what variables most affect predictions, partial dependence plots show how a feature affects predictions.

Partial dependence plots can be interpreted similarly to coefficients in linear or logistic regression models, but can capture more complex patterns than simple coefficients.

We can use partial dependence plots to answer questions like :
- Controlling for all other house features, what impact do longitude and latitude have on home prices? To restate this, how would similarly sized houses be priced in different areas?
- Are predicted health differences between two groups due to differences in their diets, or due to some other factor?

### How does it work?

Partial dependence plots are calculated after a model has been fit. How do we then disentangle the effects of several features?

We start by selecting a single row. We will use the fitted model to predict our outcome of that row. But we repeatedly **alter the value** for **one variable** to make a series of predictions.

For example, in the football example used above, we could predict the outcome if the team had the ball 40% of the time, but also 45, 50, 55, 60, ...

We build the plot by:
- representing on the horizontal axis the value change in the ball possession for example
- and on the horizontal axis the change of the outcome

We don't use only a single row, but many rows to do that. Therefore, we can represent a confidence interval and an average value, just like on this graph:

![image](https://maelfabien.github.io/assets/images/perm3.jpg)

The blue shaded area indicates the level of condifence.

### Example

Back to our FIFA Man of the Game example :

```python
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestClassifier
from sklearn.tree import DecisionTreeClassifier

data = pd.read_csv('../input/fifa-2018-match-statistics/FIFA 2018 Statistics.csv')

y = (data['Man of the Match'] == "Yes")  # Convert from string "Yes"/"No" to binary
feature_names = [i for i in data.columns if data[i].dtype in [np.int64]]
X = data[feature_names]

X_train, X_test, y_train, y_test = train_test_split(X, y, random_state=1)
tree_model = DecisionTreeClassifier(random_state=0, max_depth=5, min_samples_split=5).fit(X_train, y_train)
```

Then, we can plot the Partial Dependence Plot using [PDPbox](https://pdpbox.readthedocs.io/en/latest/). The goal of this library is to visualize the impact of certain features towards model prediction for any supervised learning algorithm using partial dependence plots. The PDP fot the number of goals scored is the following :

```python
from matplotlib import pyplot as plt
from pdpbox import pdp, get_dataset, info_plots

# Create the data that we will plot
pdp_goals = pdp.pdp_isolate(model=tree_model, dataset=X_test, model_features=feature_names, feature='Goal Scored')

# plot it
pdp.pdp_plot(pdp_goals, 'Goal Scored')
plt.show()
```

![image](https://maelfabien.github.io/assets/images/perm4.jpg)

From this particular graph, we see that scoring a goal substantially increases your chances of winning "Man of The Match." But extra goals beyond that appear to have little impact on predictions.

We can pick a more complex model and another feature to illustrate the changes :

```python
# Build Random Forest model
rf_model = RandomForestClassifier(random_state=0).fit(X_train, y_train)

pdp_dist = pdp.pdp_isolate(model=rf_model, dataset=X_test, model_features=feature_names, feature=feature_to_plot)

pdp.pdp_plot(pdp_dist, feature_to_plot)
plt.show()
```

![image](https://maelfabien.github.io/assets/images/perm5.jpg)

### 2D Partial Dependence Plots

We can also plot interactions between features on a 2D graph.

```python
# Similar to previous PDP plot except we use pdp_interact instead of pdp_isolate and pdp_interact_plot instead of pdp_isolate_plot

features_to_plot = ['Goal Scored', 'Distance Covered (Kms)']

inter1  =  pdp.pdp_interact(model=tree_model, dataset=X_test, model_features=feature_names, features=features_to_plot)

pdp.pdp_interact_plot(pdp_interact_out=inter1, feature_names=features_to_plot, plot_type='contour')
plt.show()
```

![image](https://maelfabien.github.io/assets/images/perm6.jpg)

In this example, each feature can only take a limited number of values. What happens if we have continuous variables ? The level frontiers bring value on the interaction between the 2 variables.

![image](https://maelfabien.github.io/assets/images/perm7.jpg)

## 4. Shapley Values

We have seen so far techniques to extract general insights from a machine learning model. What if you want to break down how the model works for an individual prediction?

SHAP Values (an acronym from SHapley Additive exPlanations) break down a prediction to show the impact of each feature.

This could be used for :
- banking automatic decisiom making 
- healthcare risk factor assessment for a single person

In summary, we use SHAP values to explain individual predictions.

### How does it work ?

SHAP values interpret the impact of having a certain value for a given feature in comparison to the prediction we'd make if that feature took some baseline value.

In our football example, we could wonder how much was a prediction driven by the fact that the team scored 3 goals, instead of some baseline number of goals?

We can decompose a prediction with the following equation:

`sum(SHAP values for all features) = pred_for_team - pred_for_baseline_values`

The SHAP Value can be represented visually as follows :

![image](https://maelfabien.github.io/assets/images/shap.png)

The output value is 0.70. This is the prediction for the selected team. The base value is 0.4979. Feature values causing increased predictions are in pink, and their visual size shows the magnitude of the feature's effect. Feature values decreasing the prediction are in blue. The biggest impact comes from Goal Scored being 2. Though the ball possession value has a meaningful effect decreasing the prediction.

If you subtract the length of the blue bars from the length of the pink bars, it equals the distance from the base value to the output.

### Example

We will use the [SHAP library](https://github.com/slundberg/shap). As previously, we import the Football game example. We will look at SHAP values for a single row of the dataset (we arbitrarily chose row 5).

```python
import shap  # package used to calculate Shap values

row_to_show = 5
data_for_prediction = X_test.iloc[row_to_show]  # use 1 row of data here. Could use multiple rows if desired
data_for_prediction_array = data_for_prediction.values.reshape(1, -1)

# Create object that can calculate shap values
explainer = shap.TreeExplainer(my_model)

# Calculate Shap values
shap_values = explainer.shap_values(data_for_prediction)
```

The `shap_values` is a list with two arrays. It's cumbersome to review raw arrays, but the shap package has a nice way to visualize the results.

```python
shap.initjs()
shap.force_plot(explainer.expected_value[1], shap_values[1], data_for_prediction)
```

![image](https://maelfabien.github.io/assets/images/shap_2.png)

The output prediction is 0.7, which means that the team is 70% likely to have a player win the award.

If we take many explanations such as the one shown above, rotate them 90 degrees, and then stack them horizontally, we can see explanations for an entire dataset:

```python
# visualize the training set predictions
shap.force_plot(explainer.expected_value, shap_values, X)
```

![image](https://maelfabien.github.io/assets/images/shap_7.png)

So far, we have used `shap.TreeExplainer(my_model)`. The package has other explainers for every type of model :
- `shap.DeepExplainer` works with Deep Learning models.
- `shap.KernelExplainer` works with all models, though it is slower than other Explainers and it offers an approximation rather than exact Shap values.

### Advanced uses of SHAP Values

### Summary plots

Permutation importance creates simple numeric measures to see which features mattered to a model. But it doesn't tell you how each features matter. If a feature has medium permutation importance, that could mean it has :
- a large effect for a few predictions, but no effect in general, or
- a medium effect for all predictions.

SHAP summary plots give us a birds-eye view of feature importance and what is driving it. 

![image](https://maelfabien.github.io/assets/images/shap_3.png)

Each dot has 3 characteristics :
- Vertical location shows what feature it is depicting
- Color shows whether the feature was high or low for that row of the dataset
- Horizontal location shows whether the effect of that value caused a higher or lower prediction

In this specific example, the model ignored `Red` and `Yellow & Red` features.High values of goal scored caused higher predictions, and low values caused low predictions.

Summary plots can be built the following way :

```python
# Create object that can calculate shap values
explainer = shap.TreeExplainer(my_model)

# Calculate shap_values for all of X_test rather than a single row, to have more data for plot.
shap_values = explainer.shap_values(X_test)

# Make plot. Index of [1] is explained in text below.
shap.summary_plot(shap_values[1], X_test)
```

Computing SHAP values can be slow on large datasets.

### SHAP Dependence Contribution plots

Partial Dependence Plots to show how a single feature impacts predictions. But they don't show the distribution of the effects for example. 

![image](https://maelfabien.github.io/assets/images/shap_4.png)

Each dot represents a row of data. The horizontal location is the actual value from the dataset, and the vertical location shows what having that value did to the prediction. The fact this slopes upward says that the more you possess the ball, the higher the model's prediction is for winning the Man of the Match award.

The spread suggests that other features must interact with Ball Possession %. For the same ball possession, we encounter SHAP values that range from -0.05 to 0.07.

![image](https://maelfabien.github.io/assets/images/shap_5.png)

We can also notice outliers that stand out spatially as being far away from the upward trend.

![image](https://maelfabien.github.io/assets/images/shap_6.png)

We can find an interpretation for this : In general, having the ball increases a team's chance of having their player win the award. But if they only score one goal, that trend reverses and the award judges may penalize them for having the ball so much if they score that little.

To implement Dependence Contribution plors, we can use the following code :


```python
# Create object that can calculate shap values
explainer = shap.TreeExplainer(my_model)

# calculate shap values. This is what we will plot.
shap_values = explainer.shap_values(X)

# make plot.
shap.dependence_plot('Ball Possession %', shap_values[1], X, interaction_index="Goal Scored")
```

### Summary plots

We can also just take the mean absolute value of the SHAP values for each feature to get a standard bar plot (produces stacked bars for multi-class outputs):

```python
shap.summary_plot(shap_values, X, plot_type="bar")
```

![image](https://maelfabien.github.io/assets/images/shap_8.png)

### Interaction plots

We can represent the interaction effect for two features and the effect on the SHAP Value they have. This can be done by plotting a dependence plot between of the interaction values. Let's take another random example in which we consider the interaction between the age and the white blood cells, and the effect this has on the SHAP interaction values :

```
shap.dependence_plot(
("Age", "White blood cells"),
shap_interaction_values, X.iloc[:2000,:],
display_features=X_display.iloc[:2000,:]
)
```

![image](https://maelfabien.github.io/assets/images/shap_9.png)