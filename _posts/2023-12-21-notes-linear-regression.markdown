---
layout: single
title:  "Notes on linear regression"
date:   2023-12-21 15:00:00 +0800
---
## Overview

I'm reviewing core statistics and machine learning concepts and this is a compilation of some notes that I've been taking:

## Linear Regression

The linear regression model can be expressed as

$$y = XW + \epsilon$$

Where $$X$$ is our input matrix, with dimensions $$n$$ x $$m$$ ($$n$$ = number of rows, $m$ = number of features), $$W$$ is our weight matrix for our regression, and $$y$$ is our output vector. We add a vector of 1s to our X so that we can add our weight term, $$w0$$, into our weight matrix $$W$$, otherwise we would have to add an intercept term $$b$$ and thus have $$y = XW + b + \epsilon$$.

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

Questions:

1. What are the cases where we could find that $$X^{T}X$$ is invertible?
   The matrix $$X^{T}X$$ may be invertible in cases of simple problems, where the dataset is not very large and the values of X allow for $$X^{T}X$$ to be inverted.

2. What do we do if $$X^{T}X$$ is invertible?
   If it is invertible, great! That means that your linear regression problem can be solved in a single step. By simply computing the values above, you can find the optimal weights $$W$$ for your problem. This has two key advantages:

- Simple and interpretable: The normal equation provides a clear interpretation of the relationship between predictor variables and target values.
- Closed-form solution: It allows you to obtain the regression coefficients without iterative optimization methods like gradient descent.

3. What are some problems with calculating $$X^{T}X$$? Even if in theory, $$X^{T}X$$ is possible, are there some problems that can still arise when calculating it?
   When calculating $$X^{T}X$$, several problems can arise, even if theoretically the value of $$X^{T}X$$ is possible (i.e., you can calculate it on pen and paper):

   - Computational complexity: calculating $$X^{T}X$$ can be really expensive. The calculation for the matrix inversion is generally $$O(n^{3})$$, making it infeasible for large datasets. In addition, matrix inversion requires storing both the original matrix ($$X^{T}X$$) and the inverted matrix ($$(X^{T}X)^{-1}$$) in memory, which can take up large amounts of RAM.
   - Numerical instability: computers are imperfect with dealing with floating point logic (see [this](https://en.wikipedia.org/wiki/Floating-point_error_mitigation) Wikipedia article about floating point errors). When the program is dealing with many significant digits of decimal points or with small decimal point numbers, it will likely make some errors, and these errors can compound and cascade, leading to your output being much different than what you would expect.

    These are two cases where, even if technically $$X^{T}X$$ can be found (i.e., you can calculate it on pen and paper), a computer software might not be able to calculate it

4. When can the matrix $$X^{T}X$$ be non-invertible?

There are some cases where we aren't going to be able to invert $$X^{T}X$$, which can be due to the underlying nature of the data itself.

- For example, we may have a **singular matrix**, which is a square matrix that does not have an inverse (i.e., there is no $$B$$ such that $$XB = I$$, so no matrix exists that if it is multiplied with $$X$$, we receive the identity matrix):
  - To check if a square matrix is singular, you have to check if the **determinant** is 0 ([what is a determinant?](https://en.wikipedia.org/wiki/Determinant)). To calculate the determinant, you can either use [cofactor expansion](https://people.math.carleton.ca/~kcheung/math/notes/MATH1107/wk07/07_cofactor_expansion.html) or simply use `np.linalg.det(array)`.
    - What does a **determinant** tell you?
    A determinant tells you how much the matrix "stretches" space. Imagine, for example, having a square grid, and then multiplying a matrix onto that square grid. To what degree is that square grid *stretched*? [Here](https://www.youtube.com/watch?v=Ip3X9LOh2dk) is a really great 3Blue1Brown visualization of what this looks like.
    - With respect to inverse matrices, the inverse of $$X$$, $$X^{-1}$$, can be found by using: $$X^{-1} = \frac{adj(X)}{det(X)}$$, where $$adj(X)$$ is the adjoint/adjugate matrix ([what is the adjoint/adjugate matrix?](https://www.cuemath.com/algebra/adjoint-of-a-matrix/))
  - If a matrix has a determinant of 0, then we can't find the inverse $$X^{-1}$$ since we can't divide by zero. Conceptually, this would also not be possible since a determinant of 0 means that the matrix "collapses" space to 0 when it stretches it.

  In short, we can check if we have a singular matrix for $$X^{T}X$$, and if we do, then we will not be able to invert it.
- s

5. What can we do if the matrix $$X^{T}X$$ is not invertible?
