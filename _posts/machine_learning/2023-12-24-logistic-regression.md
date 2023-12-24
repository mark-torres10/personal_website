---
layout: single
title: "Detecting credit card fraud using machine learning: Part I"
date: 2023-12-24 14:00:00 +0800
classes: wide
toc: true
categories:
  - machine_learning
permalink: /machine_learning/detecting-fraud-using-ml-pt1
---

# Detecting credit card fraud using machine learning: Part I

In this example, we'll create a model that will help us detect credit card fraud. This is a classic application of machine learning that is illustrative of the type of work that goes into creating a useful machine learning model. We will then host this as an API endpoint using Google Cloud (in Part II). We'll also create an end-to-end ML pipeline for model training, experimentation, deployment, and monitoring (using [this repo](https://github.com/DataTalksClub/mlops-zoomcamp) as a running resource) (in Part III).

## Motivation

Fraud detection is a classic example of a problem solved using classification algorithms. It's a good case study in basic machine learning development for the following reasons:

1. Real-world applications: Credit card fraud detection is a real-world problem.
2. Feature engineering: To make credit card data useful, it has to be transformed and manipulatedin various ways.
3. Imbalanced datasets: credit card fraud is (thankfully) a relatively rare occurrence, so detecting fraud requires managing imbalanced datasets.
4. Interpretability: since this problem is within a non-technical domain (finance), working on this project in industry will likely require talking with non-ML people. These people will likely be very interested in not only a model that can predict fraud, but also what the model looks for when it detects fraud. Therefore, we want a model that is interpretable.

## Setup and loading data

For our data, we'll be using [this](https://www.kaggle.com/datasets/mishra5001/credit-card/data) dataset from Kaggle, which is a sample dataset for credit card fraud detection.

Let's get our data loaded as well as import any missing packages

```python
import pandas as pd

pd.set_option('display.max_rows', 5)
pd.set_option('display.max_columns', 5)
pd.set_option('display.max_colwidth', None)
pd.set_option('display.width', None)
```

```python
df = pd.read_csv("application_data.csv")
```

## Data Exploration

Let's now take a quick look at our data

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
      <th>SK_ID_CURR</th>
      <th>TARGET</th>
      <th>...</th>
      <th>AMT_REQ_CREDIT_BUREAU_QRT</th>
      <th>AMT_REQ_CREDIT_BUREAU_YEAR</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>100002</td>
      <td>1</td>
      <td>...</td>
      <td>0.0</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>100003</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>100004</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>100006</td>
      <td>0</td>
      <td>...</td>
      <td>NaN</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>4</th>
      <td>100007</td>
      <td>0</td>
      <td>...</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 122 columns</p>
</div>

What features do we have available in our data? We can look at the `columns_description.csv` file in order to see what the features are.

```python
column_descriptions = pd.read_csv("columns_description.csv")
```

This describes the features in our dataset. For our use case, we'll only look at the data in `application_data.csv`.

```python
column_descriptions = column_descriptions[column_descriptions['Table'] == "application_data"][["Row", "Description", "Special"]]
```

```python
column_descriptions.head(100)
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
      <th>Row</th>
      <th>Description</th>
      <th>Special</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>SK_ID_CURR</td>
      <td>ID of loan in our sample</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>TARGET</td>
      <td>Target variable (1 - client with payment difficulties: he/she had late payment more than X days on at least one of the first Y installments of the loan in our sample, 0 - all other cases)</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>98</th>
      <td>FLAG_DOCUMENT_4</td>
      <td>Did client provide document 4</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>99</th>
      <td>FLAG_DOCUMENT_5</td>
      <td>Did client provide document 5</td>
      <td>NaN</td>
    </tr>
  </tbody>
</table>
<p>100 rows × 3 columns</p>
</div>

## Data Preprocessing

## Dealing with Data Imbalances

## Model Development

## Model Evaluation and Iteration

## Model Interpretation

## Summary and Next Steps
