---
layout: single
title:  "Predicting house prices using linear regression: an example"
date:   2023-12-22 17:00:00 +0800
classes: wide
toc: true
categories:
- machine_learning
permalink: /machine_learning/predicting-house-prices-linear-regression
---
# Predicting housing prices using linear regression

We are using linear regression in order to predict housing prices from an example Kaggle dataset.

This is a part of the [ML Zoomcamp](https://github.com/DataTalksClub/machine-learning-zoomcamp/), a guided set of tutorials on teaching machine learning.

Source: https://github.com/DataTalksClub/machine-learning-zoomcamp/blob/master/cohorts/2023/02-regression/homework.md



```python
# we grab the data 
# !wget https://raw.githubusercontent.com/alexeygrigorev/datasets/master/housing.csv
```

    --2023-12-22 16:29:37--  https://raw.githubusercontent.com/alexeygrigorev/datasets/master/housing.csv
    Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 2606:50c0:8001::154, 2606:50c0:8000::154, 2606:50c0:8003::154, ...
    Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|2606:50c0:8001::154|:443... connected.
    HTTP request sent, awaiting response... 200 OK
    Length: 1423529 (1.4M) [text/plain]
    Saving to: ‘housing.csv’
    
    housing.csv         100%[===================>]   1.36M  2.14MB/s    in 0.6s    
    
    2023-12-22 16:29:38 (2.14 MB/s) - ‘housing.csv’ saved [1423529/1423529]
    


Let's get our data loaded as well as import any necessary packages



```python
import matplotlib.pyplot as plt
import numpy as np
import pandas as pd
import seaborn as sns

%matplotlib inline
```


```python
df = pd.read_csv("housing.csv")
```

Just taking a peek at the data that we have:



```python
df.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>longitude</th>
      <th>latitude</th>
      <th>housing_median_age</th>
      <th>total_rooms</th>
      <th>total_bedrooms</th>
      <th>population</th>
      <th>households</th>
      <th>median_income</th>
      <th>median_house_value</th>
      <th>ocean_proximity</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>-122.23</td>
      <td>37.88</td>
      <td>41.0</td>
      <td>880.0</td>
      <td>129.0</td>
      <td>322.0</td>
      <td>126.0</td>
      <td>8.3252</td>
      <td>452600.0</td>
      <td>NEAR BAY</td>
    </tr>
    <tr>
      <th>1</th>
      <td>-122.22</td>
      <td>37.86</td>
      <td>21.0</td>
      <td>7099.0</td>
      <td>1106.0</td>
      <td>2401.0</td>
      <td>1138.0</td>
      <td>8.3014</td>
      <td>358500.0</td>
      <td>NEAR BAY</td>
    </tr>
    <tr>
      <th>2</th>
      <td>-122.24</td>
      <td>37.85</td>
      <td>52.0</td>
      <td>1467.0</td>
      <td>190.0</td>
      <td>496.0</td>
      <td>177.0</td>
      <td>7.2574</td>
      <td>352100.0</td>
      <td>NEAR BAY</td>
    </tr>
    <tr>
      <th>3</th>
      <td>-122.25</td>
      <td>37.85</td>
      <td>52.0</td>
      <td>1274.0</td>
      <td>235.0</td>
      <td>558.0</td>
      <td>219.0</td>
      <td>5.6431</td>
      <td>341300.0</td>
      <td>NEAR BAY</td>
    </tr>
    <tr>
      <th>4</th>
      <td>-122.25</td>
      <td>37.85</td>
      <td>52.0</td>
      <td>1627.0</td>
      <td>280.0</td>
      <td>565.0</td>
      <td>259.0</td>
      <td>3.8462</td>
      <td>342200.0</td>
      <td>NEAR BAY</td>
    </tr>
  </tbody>
</table>
</div>



### Exploratory data

Before we start doing any machine learning, we should take a look at the data that we have, so that we can get a general sense of what we're working with. The purpose of this exploration is twofold:

1. Check the relationships that the features have with each other and with the output variable. Doing so will let us see if any of the features are correlated with each other as well as get an idea for how the features should correlate with the output variable.
2. Determine which features to use in our model.


We want to predict the `median_house_value` column, so let's pull that from our dataset and just see how it looks



```python
sns.histplot(df["median_house_value"], bins=50)
```




    <Axes: xlabel='median_house_value', ylabel='Count'>




    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_9_1.png)
    


It looks like the distribution has a heavy right skew. The large number of values at 500,000 is likely a matter of coding, where anything >$500,000 was just coded as 500,000.

Let's remove these from our dataset as we don't want those arbitrarily coded values to affect our dataset (plus, 500,000 isn't actually what they were sold for, so if we keep them then we're doing a classification problem instead of a regression problem).

We also generally dislike skew in our output variables. Although we don't explicitly need a normal distribution for our output variable, linear regression does assume that the residuals are normally distributed (see [here](https://www.statisticssolutions.com/free-resources/directory-of-statistical-analyses/normality/) for an explanation). If our outcome variable $$Y$$ is too skewed, then it becomes more unlikely that our residuals are normally distributed, and for our residuals to be closer to normally distributed our regression outline would have to be pulled in the direction of our skew, which increases the errors on the majority of our distribution.

To resolve this, we can do a log transformation of our output variable (see [here](https://onlinestatbook.com/2/transformations/log.html) for an explanation). The log transform reduces skewness and brings values closer to the center of the distribution. We can do this using `np.log1p` (we use the `1p` since we can't take a log of a zero, so using 1p prevents that problem). We will use this transformed variable as our output $$Y$$ and then exponentiate it (using `np.exp1m`) to get our predictions.

*Note: the more transformations that we do on our dataset, the less interpretable our results are going to be (even though it is a linear regression). We need to keep this in mind when making our modeling choices - do we want to prioritize performance at the cost of simplicity and explainability. Notably, a linear regression is probably the most explainable ML algorithm, but as we have the option to choose other ML algorithms we need to also consider this explainability vs. black box tradeoff as well.*

So, in short, we will:
1. Remove all the rows whose `median_house_value` was coded as the maximum value of 500,000 (we might lose some homes that actually were 500,000, but the vast majority are homes that were priced at >500,000) (note: if we wanted to account for the homes that were >500,000, we could build a second model, a classification model, that determines if a home was sold for =<500,000 or >500,000).
2. Perform a log transformation on the y-values in order to reduce skew.



```python
max_house_value = df["median_house_value"].max()
```


```python
num_max_coded_rows = len(
    [
        val for val in df["median_house_value"].values
        if val == max_house_value
    ]
)
```


```python
print(f"Number of rows with max value that we will be removing: {num_max_coded_rows}")
print(f"Dataframe length before filtering: {len(df)}")
```

    Number of rows with max value that we will be removing: 965
    Dataframe length before filtering: 20640



```python
# exclude all rows where median_house_value is equal to the max value
df = df[df["median_house_value"] != max_house_value]
```


```python
print(f"Dataframe length after filtering: {len(df)}")
```

    Dataframe length after filtering: 19675


Let's take a look at the distribution after removing the 500,000+ values:


```python
sns.histplot(df["median_house_value"], bins=50)
```




    <Axes: xlabel='median_house_value', ylabel='Count'>




    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_17_1.png)
    


Now the spike in the values after 500,000 is gone, but there is still a right skew. Let's log-transform the y-values in order to fix this.


```python
median_housing_values = df["median_house_value"]
y = np.log1p(df["median_house_value"])
```


```python
sns.histplot(y, bins=50)
```




    <Axes: xlabel='median_house_value', ylabel='Count'>




    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_20_1.png)
    


This still has some skew, but it is a big improvement over our previous output variable. The lower the skewness of the underlying distribution, the more likely it is that the residuals will actually follow a normal distribution, which is a fundamental assumption of regression.


```python
del df["median_house_value"]
```

For our problem, let's take a look at only the properties that are not right on the water, but are some distance away. Since this is a linear regression problem, let's also just look at the columns that are numeric and might have some relationship with the median home price.


```python
property_types = ["<1H OCEAN", "INLAND"]
X = df[df['ocean_proximity'].isin(property_types)]
y = y[y.index.isin(X.index)]
columns = [
    "housing_median_age",
    "total_rooms",
    "total_bedrooms",
    "population",
    "households", 
    "median_income"
]

X = X[columns]
```

Let's take a look at our dataset now and see what we can learn:


```python
X.head()
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>housing_median_age</th>
      <th>total_rooms</th>
      <th>total_bedrooms</th>
      <th>population</th>
      <th>households</th>
      <th>median_income</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>701</th>
      <td>32.0</td>
      <td>1283.0</td>
      <td>194.0</td>
      <td>485.0</td>
      <td>171.0</td>
      <td>6.0574</td>
    </tr>
    <tr>
      <th>830</th>
      <td>9.0</td>
      <td>3666.0</td>
      <td>711.0</td>
      <td>2341.0</td>
      <td>703.0</td>
      <td>4.6458</td>
    </tr>
    <tr>
      <th>859</th>
      <td>21.0</td>
      <td>4342.0</td>
      <td>783.0</td>
      <td>2172.0</td>
      <td>789.0</td>
      <td>4.6146</td>
    </tr>
    <tr>
      <th>860</th>
      <td>15.0</td>
      <td>3575.0</td>
      <td>597.0</td>
      <td>1777.0</td>
      <td>559.0</td>
      <td>5.7192</td>
    </tr>
    <tr>
      <th>861</th>
      <td>20.0</td>
      <td>4126.0</td>
      <td>1031.0</td>
      <td>2079.0</td>
      <td>975.0</td>
      <td>3.6832</td>
    </tr>
  </tbody>
</table>
</div>



When doing exploratory data analysis, some of the things that we want to look for include:

1. Missing data: do we have any data that is missing? If so, can we impute the missing data?
2. Correlations: are any of the features correlated to each other? If so, can we eliminate redundant features?

Let's do these in steps.

*Note: we do NOT need to scale data when it comes to linear regression. This is because each parameter estimate is scaled to each feature already. For regression, we would only scale if we want our intercepts to be centered at 0 for interpretability purposes, but that doesn't apply in this case. We need to scale if we're working with algorithms that use distance metrics of some sort, such as clustering algorithms or gradient descent in neural networks (see [here](https://www.atoti.io/articles/when-to-perform-a-feature-scaling/) for more information)*

#### Missing data

We want to check if there is any data that is missing in our dataset. We can look for the presence of NaN values and similar values such as null or None. We also have to be careful for the presence of other values that can signify missing data. For example, when working with survey data sometimes a field is coded as -999 if there is no valid answer. Sometimes we might even see empty strings, "", used to denote missing data. In this case, given our feature set, looking for NaN values should suffice, especially since the fields that we're looking at are already all converted to numerics (meaning that any non-numeric value is encoded as NaN).


```python
fields_with_missing_data = []

for field in X.columns:
    missing_data = X[field].isnull().sum()
    if missing_data > 0:
        fields_with_missing_data.append(field)
        print(f"{field} has {missing_data} missing values")
```

    total_bedrooms has 154 missing values


We see that `total_bedrooms` has some missing values whereas the rest of the data does not have any missing values. We may want to somehow impute values for the missing values in `total_bedrooms`, but let's first determine if this is a feature that we want to keep in the first place, which we can find out if we look at the feature correlations.

#### Correlations

We want features that are generally not really correlated with each other. Conceptually, this is because we want each feature used in our prediction to add more information that will help us make a better prediction.

*Note: mathematically, we need features that aren't correlated - the problem of correlated features, also known as multicollinearity, makes the linear regression problem more difficult, see [here](https://markptorres.com/machine_learning/notes-linear-regression) for more information*

Let's take a look at the correlations between our features:

Let's take a look at the correlations of our features


```python
sns.heatmap(X.corr(), annot=True)
```




    <Axes: >




    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_33_1.png)
    


As we can see here, the `total_rooms`, `total_bedrooms`, `population`, and `households` are really highly correlated with each other. If we think about it, this makes sense - the larger the population, the more households there are. The more households there are, the more total bedrooms there are, and the more total bedrooms there are, the more total rooms there are.

Because of this, we can choose to just keep one of the variables. Since our purpose is to predict house prices, it makes logical sense that the price of a house is likely highly related to how large the house is, which is a factor of the total rooms in the house (of which bedrooms is a part), which leaves us with `total_rooms` and `total_bedrooms`. Since these two are highly correlated (and logically so), we should pick only one. Since we found that `total_bedrooms` has some rows with missing values, let's use `total_rooms` instead.

#### Picking a subset of features and mapping them against our output variable

After exploring our features, we now have a sense of which features we should include in our model. We want to pick features that are not correlated with each other, so that each can help us predict the home price. Let's choose the `housing_median_age`, `median_income`, and `total_rooms` variables. Now let's check how these each relate to our log-transformed output variable, `housing_median_price`.


```python
cols = ["housing_median_age", "median_income", "total_rooms"]
X = X[cols]
```


```python
data = X.copy()
data["median_house_value"] = y.values
```


```python
# show the correlation matrix between each column in X and the values in y
sns.heatmap(data.corr(), annot=True)
```




    <Axes: >




    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_38_1.png)
    


From the looks of it, `median_income` is the most correlated to `median_house_price`. Logically this makes sense - the higher a community's income, the more expensive the homes in the community likely are. We also see that `housing_median_age` is not correlated with `median_housing_price`, which also makes sense - it's not the age of the homebuyer that matters but rather the features of the home itself (like the `total_rooms`, which has some slight correlation).

Let's also create scatterplots of each feature against the output variable just to see how they look. We'll plot the features against their original prices, since this will look cleaner from a visualization perspective and there will be less cluttering as opposed to the log-transformed version of the prices:


```python
for col in X.columns:
    sns.scatterplot(x=X[col], y=np.expm1(y))
    plt.show()
```


    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_41_0.png)
    



    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_41_1.png)
    



    
![png](assets/images/2023-12-22-linear-regression-example_files/2023-12-22-linear-regression-example_41_2.png)
    


The scatterplots reinforce what we already observed from the correlation matrix and give us some indication of how each feature variable relates to the median home price. We can later superimpose the coefficients created by our regression model against these scatterplots to see how well the coefficients for each parameter reflect the underlying data.

Now that we've done these exercises, we now know (1) which features we want to use and (2) how these features relate to our output variable, `median_house_price`. This means that we are now ready to actually perform the regression.

## Perform regression

In this overview, we'll skip over [what is a linear regression](https://www.ibm.com/topics/linear-regression), [how to solve the linear regression equation](https://markptorres.com/machine_learning/notes-linear-regression) as well as [why linear regression minimizes squared error](https://markptorres.com/machine_learning/notes-least-squares-regression) and we'll instead focus on implementation. To perform our regression, we need to do the following:

1. Split up our data into training, validation, and test sets.
2. Create regression model.
3. Evaluate our model's performance.

*Note: we are sticking to a linear regression for this walkthrough. We will not explore techniques such as regularization or dimensionality reduction. We also will not be doing cross-validation, though [this Kaggle tutorial](https://www.kaggle.com/code/jnikhilsai/cross-validation-with-linear-regression) has a great walkthrough of doing cross validation with linear regression*

### Splitting our data into training, validation, and test sets

We want to split up our data into training, validation, and test sets.

- Training: this is the data that our model receives and is trained on.
- Validation: this is the data that we use to evaluate our model's performance and to tune our model. We use this to see how well our current iteration of the model is doing and compare that to other versions of our model. This comes in handy, for example, if we're tuning our hyperparameters, changing algorithm type, etc.
- Test: this is a set of data that we use to evaluate our final model. This is never used during the model creation and iteration process and is only used at the end to actually see.

### Creating a regression model

### Evaluating our model's performance

#### Spot-checking a few examples

#### Evaluating error metrics

#### Plotting the regressions coefficients against the scatterplots of each feature against the output variable

## Checking the assumptions of linear regression

Now that we have a working model, we need to check to see if our model does fit the assumptions of linear regression.


