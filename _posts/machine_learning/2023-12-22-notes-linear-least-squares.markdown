---
layout: single
title:  "Notes on least squares regression"
date:   2023-12-22 15:00:00 +0800
classes: wide
toc: true
categories:
- machine_learning
permalink: /machine_learning/notes-least-squares-regression
---

## Overview

## Goals of our linear function

We want a linear function that fulfills the following two constraints:

1. We find a weight vector, $$W$$, such that:
    $$XW\approx y$$
2. The squared prediction error is minimized:
    $$\sum_{i=1}^{n}(y_i - \vec{x_i}\vec{w_i})^{2}$$

   This is known as the **residual sum of squares**.

For part (1), we can derive that the optimal solution for $$W$$ is $$(X^{T}X)^{-1}X^{T}y$$, in the following way:
$$XW=y$$
It is likely that X is not invertible (otherwise, we multiply by $$X^{-1}$$ on both sides). Therefore, we multiply by $$X^{T}$$ in order to create a (hopefully) invertible matrix, $$X^{T}X$$
$$X^{T}XW=X^{T}y$$
Now we invert this matrix,
$$(X^{T}X)^{-1}X^{T}XW = (X^{T}X)^{-1}X^{T}y$$
$$IW = (X^{T}X)^{-1}X^{T}y$$
$$W = (X^{T}X)^{-1}X^{T}y$$

However, we can show that this solution for W is also one that fulfills condition (2), minimizing the squared prediction error.

## Proof: Why does the closed form solution to $$XW=y$$ minimize the residual sum of squares?

We can use calculus in order to demonstrate that this solution does minimize the residual sum of squares.

Some things to note first:

1. $$a^{T} \cdot b$$ = $$b^{T} \cdot a$$ (since it is just the sum of elementwise products, so order doesn't matter).
2. If we have a function $$f(x)$$ that takes as input a scalar $$x$$, then $$\frac{d}{dx}$$ is the derivatve of the function with respect to $$x$$. If we instead have the same function $$f(x)$$ but it takes as input a vector $$\vec{x}$$, then the derivative of the function is now the **gradient**, and the gradient is just a vector of the derivatives with respect to each component of $$\vec{x}$$:

$$\nabla{f} \triangleq \begin{bmatrix}
\frac{\partial f}{\partial x_1} \\ \frac{\partial f}{\partial x_2} \\ ... \\ \frac{\partial f}{\partial x_n}
\end{bmatrix}$$

(see [here](https://pillowlab.princeton.edu/teaching/statneuro2018/slides/notes03b_LeastSquaresRegression.pdf) for more notes)
