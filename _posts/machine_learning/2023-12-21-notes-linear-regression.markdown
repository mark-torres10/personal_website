---
layout: single
title: "Notes on linear regression"
date: 2023-12-21 15:00:00 +0800
classes: wide
toc: true
categories:
  - machine_learning
permalink: /machine_learning/notes-linear-regression
---

## Overview

I'm reviewing core statistics and machine learning concepts and this is a compilation of some notes that I've been taking.

## Linear Regression Theory

The linear regression model can be expressed as

$$y = XW + \epsilon$$

Where $$X$$ is our input matrix, with dimensions $$n$$ x $$m$$ ($$n$$ = number of rows, $$m$$ = number of features), $$W$$ is our weight matrix for our regression, and $$y$$ is our output vector. We add a vector of 1s to our X so that we can add our weight term, $$w_0$$, into our weight matrix $$W$$, otherwise we would have to add an intercept term $$b$$ and thus have $$y = XW + b + \epsilon$$.

We need to solve for $$W$$ to get our weight matrix. If X is invertible, the solution is simple:

$$X^{-1}y = X^{-1}XW$$
$$X^{-1}y = IW = W$$

But, it's unlikely that X will be invertible. This is because only square matrices are invertible, and it's likely that X will have many more rows (observations) than columns (features).

Another approach we can use is taking $$X^{T}X$$, which is a square matrix.

$$y = XW$$
$$X^{T}y = X^{T}XW$$
$$(X^{T}X)^{-1}X^{T}y = (X^{T}X)^{-1}X^{T}XW = IW = W$$
$$(X^{T}X)^{-1}X^{T}y = W$$

This is called the "normal equation" of linear regression. If $$X^{T}X$$ is invertible, then we can find, in a single step, the optimal weights $$W$$ that solves the equation $$XW = y$$

### What are the cases where we could find that $$X^{T}X$$ is invertible?

The matrix $$X^{T}X$$ may be invertible in cases of simple problems, where the dataset is not very large and the values of X allow for $$X^{T}X$$ to be inverted.

### What do we do if $$X^{T}X$$ is invertible?

If it is invertible, great! That means that your linear regression problem can be solved in a single step. By simply computing the values above, you can find the optimal weights $$W$$ for your problem. This has two key advantages:

- Simple and interpretable: The normal equation provides a clear interpretation of the relationship between predictor variables and target values.
- Closed-form solution: It allows you to obtain the regression coefficients without iterative optimization methods like gradient descent.

### What are some problems with calculating $$X^{T}X$$? Even if in theory, $$X^{T}X$$ is possible, are there some problems that can still arise when calculating it?

When calculating $$X^{T}X$$, several problems can arise, even if theoretically the value of $$X^{T}X$$ is possible (i.e., you can calculate it on pen and paper):

- Computational complexity: calculating $$X^{T}X$$ can be really expensive. The calculation for the matrix inversion is generally $$O(n^{3})$$, making it infeasible for large datasets. In addition, matrix inversion requires storing both the original matrix ($$X^{T}X$$) and the inverted matrix ($$(X^{T}X)^{-1}$$) in memory, which can take up large amounts of RAM.
- Numerical instability: computers are imperfect with dealing with floating point logic (see [this](https://en.wikipedia.org/wiki/Floating-point_error_mitigation) Wikipedia article about floating point errors). When the program is dealing with many significant digits of decimal points or with small decimal point numbers, it will likely make some errors, and these errors can compound and cascade, leading to your output being much different than what you would expect.

These are two cases where, even if technically $$X^{T}X$$ can be found (i.e., you can calculate it on pen and paper), a computer software might not be able to calculate it

### When can the matrix $$X^{T}X$$ be non-invertible?

When the matrix $$X^{T}X$$ is non-invertible, it means that the coefficients derived from regression will be unstable (i.e., you run it two times and you might get vastly different coefficient values, or you perturb the input values slightly and the coefficient values change dramatically).

There are some cases where we aren't going to be able to invert $$X^{T}X$$, which can be due to the underlying nature of the data itself.

#### Singular matrix

We may have a **singular matrix**, which is a square matrix that does not have an inverse (i.e., there is no $$B$$ such that $$XB = I$$, so no matrix exists that if it is multiplied with $$X$$, we receive the identity matrix):

To check if a square matrix is singular, you have to check if the **determinant** is 0 ([what is a determinant?](https://en.wikipedia.org/wiki/Determinant)). To calculate the determinant, you can either use [cofactor expansion](https://people.math.carleton.ca/~kcheung/math/notes/MATH1107/wk07/07_cofactor_expansion.html) or simply use `np.linalg.det(array)`.

- What does a **determinant** tell you?
  A determinant tells you how much the matrix "stretches" space. Imagine, for example, having a square grid, and then multiplying a matrix onto that square grid. To what degree is that square grid _stretched_? [Here](https://www.youtube.com/watch?v=Ip3X9LOh2dk) is a really great 3Blue1Brown visualization of what this looks like.

- With respect to inverse matrices, the inverse of $$X$$, $$X^{-1}$$, can be found by using: $$X^{-1} = \frac{adj(X)}{det(X)}$$, where $$adj(X)$$ is the adjoint/adjugate matrix ([what is the adjoint/adjugate matrix?](https://www.cuemath.com/algebra/adjoint-of-a-matrix/))

- If a matrix has a determinant of 0, then we can't find the inverse $$X^{-1}$$ since we can't divide by zero. Conceptually, this would also not be possible since a determinant of 0 means that the matrix "collapses" space to 0 when it stretches it.

##### When can you expect a matrix to be singular?

When $$X^{T}X$$ is singular, this can be signs of two things:

1. Perfect multicollinearity: one or more of the columns in $$X$$ is a linear combination of other columns. This is an example of **multicollinearity**, where one or more columns is a linear combination of other columns (i.e., not all the columns are unique). This is not good for regression since we will be unable to come up with unique solutions (specifically, since the column that is a linear combination of other columns can be expressed with infinitely different combinations (see [here](https://statisticsbyjim.com/regression/multicollinearity-in-regression-analysis/) for a more detailed explanation)).

2. Underdetermined system: When you have more factors than you do observations (i.e,. $$m$$ > $$n$$, or more columns than rows), then you have an **underdetermined system**, where there aren't enough rows to estimate unique coefficients since you have more unknown variables than you do equations (see [here](https://en.wikipedia.org/wiki/Underdetermined_system) for an explanation).

In short, we can check if we have a singular matrix for $$X^{T}X$$, and if we do, then we will not be able to invert it.

#### Ill-conditioned matrix

We may have an **ill-conditioned matrix** (see [here](https://www.math.wsu.edu/math/faculty/tsat/teach/files/230/c230.pdf) for an explanation) that is technically invertible, but quickly becomes non-invertible/singular if the inputs are changed ever so slightly. We can check the possibility of a matrix being ill-conditioned in several ways:

##### Condition number

We can determine the propensity of a matrix to be ill-conditioned through calculating its **condition number** (see [here](https://en.wikipedia.org/wiki/Condition_number) for a definition). Practially speaking, we can use `np.linalg.cond(X)` to calculate the condition number in Python using [numpy](https://numpy.org/doc/stable/reference/generated/numpy.linalg.cond.html). A condition number of ~1 means that the matrix $$X^{T}X$$ is likely well-conditioned and so an inverse can be found, whereas a condition number much greater than 1 means that it is likely ill-conditioned.

##### Singular value decomposition (SVD)

(see [here](https://markptorres.com/machine_learning/notes-singular-value-decomposition/) for an overview of SVD)

We can look at the singular values after performing SVD. If we look at the singular values and see any that are zero, then we cannot invert the matrix (since a zero for a singular value effectively tells us that we lose information if we pass an input through the matrix, i.e., we can't "invert" the operation (unlike how we, for example, can "invert" or "reverse" addition by doing subtraction)). Small singular values can tell us that a matrix is likely to be ill-conditioned, namely because operations that divide using the singular values, for values of $$s_i$$ close to zero, will lead to numerical instability.

##### Eigenvalues and eigenvectors

(see [here](https://markptorres.com/linear_algebra/notes-eigenvalues-eigenvectors/) for an overview of eigenvalues and eigenvectors).

Eigenvalues that are close to zero can indicate ill-conditioning. This is because eigenvalues close to zero indicate a large gap between the largest and the smallest singular values. If there is a large gap between these two values, then this gives the matrix a high condition number, which is an indicator of numerical instability (see [here](https://math.stackexchange.com/questions/4371774/condition-number-and-singular-values) and [here](https://en.wikipedia.org/wiki/Condition_number#Matrices) for more explanation).

##### Numerical instability

If, while doing the calculations for linear regression, you see numerical instability, this is a practical signal for a possibly ill-conditioned matrix.

##### Multicollinearity and numerical instability

Indications of multicollinearity: indicators of multicollinearity (e.g., high pairwise correlations between feature columns) can be indicators of possible numerical instability. High multicollinearity makes it difficult to converge to "true" and "stable" coefficient values since the slightest changes in input values can lead to wildly different coefficients. Part of the purpose of a regression is to find unique coefficients for each feature detailing the impact of that feature alone on the output variable. But, if two variables are highly correlated, then the model can assign largely arbitrary weights to either one since a decrease in one should be evened out by an increase in the other. For example, if both `num_bedrooms` and `num_rooms` (which are obviously correlated) are used in a model for predicting house price, then a model can assign either a high weight to `num_bedrooms` and a low weight to `num_rooms` or vice versa or some combination in between since any combination would have the same effect on predicting house price. For a more detailed overview of the multicollinearity problem, see [here](https://www.sciencedirect.com/topics/mathematics/multicollinearity-problem).

### Conceptually, what does the matrix $$X^{T}X$$ being invertible tell us?

#### There exists a unique linear solution to the linear regression problem

An invertible matrix tells us that there is a simple, closed-form, exact solution to the linear regression problem. Specifically, if $$X^{T}X$$ is invertible, then the weight matrix $$W$$ is simply:

$$W = (X^{T}X)^{-1}X^{T}y$$

#### Each parameter coefficient can be independently estimated and there is no linear dependence across features

A consequence of there being a unique solution to the linear regression problem is that each parameter coefficient can be independently estimated.

To better understand this, let's imagine a case where the parameter coefficients aren't independent Conceptually, if two features are highly correlated, then this means that it will be hard to determine a unique parameter coefficient for each feature.

Let's take the example of predicting home prices. If we have two features, `num_rooms` and `num_bedrooms`, these are likely highly correlated and so will make the resulting input $$X$$ matrix more likely to be ill-conditioned or singular. Conceptually, if you know what `num_rooms` already is, then knowing `num_bedrooms` likely doesn't tell you any new information that would help you predict home price, because the value of `num_bedrooms` is already highly correlated to `num_rooms`, so knowing one makes the other redundant.

### What do we do next if the matrix $$X^{T}X$$ is not invertible?

When the matrix is not invertible, then the first two things to consider are: (1) removing multicollinearity, (2) perform better feature selection, and (3) doing either regularization or dimensionality reduction (e.g., PCA).
