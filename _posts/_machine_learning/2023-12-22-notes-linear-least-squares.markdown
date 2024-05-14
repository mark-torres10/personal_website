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

## Why does the closed form solution to $$XW=y$$ minimize the residual sum of squares?

We can use calculus in order to demonstrate that this solution does minimize the residual sum of squares.

### Things to note

1. $$a^{T} \cdot b$$ = $$b^{T} \cdot a$$ (since it is just the sum of elementwise products, so order doesn't matter).
2. If we have a function $$f(x)$$ that takes as input a scalar $$x$$, then $$\frac{d}{dx}$$ is the derivatve of the function with respect to $$x$$. If we instead have the same function $$f(x)$$ but it takes as input a vector $$\vec{x}$$, then the derivative of the function is now the **gradient**, and the gradient is just a vector of the derivatives with respect to each component of $$\vec{x}$$:

    $$\nabla{f} \triangleq \begin{bmatrix}
    \frac{\partial f}{\partial x_1} \\ \frac{\partial f}{\partial x_2} \\ ... \\ \frac{\partial f}{\partial x_n}
    \end{bmatrix}$$

3. A derivative of a linear function:

    $$\frac{\partial }{\partial \vec{x}} \vec{a} \cdot \vec{x} = \vec{a} $$

    To demonstrate this:

    $$\frac{\partial }{\partial \vec{x}} \vec{a} \cdot \vec{x} = \frac{\partial}{\partial \vec{x}} \vec{a}^{T}\vec{x} =  \frac{\partial}{\partial \vec{x}} \vec{x}^{T}\vec{a} $$

    If you actually evaluate this for each element, then you find that for each element $$i$$, the dot product operation is $$x_i * a_i$$, for which the answer is $$a_i$$. Evaluating this as a derivative, we see that the result then should be $$\vec{a}$$

    This is the same principle as in single-variable calculus, where $$\frac{\partial}{\partial x} ax = a$$.

4. A derivative of a quadratic function:

    $$\frac{\partial}{\partial \vec{x}} \vec{x}^{T}A\vec{x} = 2A\vec{x}$$

    This is the same principle as in single-variable calculus, where $$\frac{\partial}{\partial x}ax^{2} = 2ax$$.

5. Our error term can be rewritten:

    We can rewrite $$\sum_{i=1}^{n}(y_i - \vec{x_i}\vec{w_i})^{2}$$ in a vectorized form: $$(\vec{Y}-\vec{X}\vec{W})^{T}(\vec{Y}-\vec{X}\vec{W})$$

### Proof

Let's take the derivative of our error function with respect to $$\vec{W}$$:

$$\frac{\partial}{\partial \vec{W}} [\sum_{i=1}^{n}(y_i - \vec{x_i}\vec{w_i})^{2}] = \frac{\partial}{\partial \vec{W}} [(\vec{Y}-\vec{X}\vec{W})^{T}(\vec{Y}-\vec{X}\vec{W})]$$

$$=\frac{\partial}{\partial \vec{W}}(Y^{T}Y - Y^{T}XW - W^{T}X^{T}Y + W^{T}X^{T}XW) $$

$$=\frac{\partial}{\partial \vec{W}}(Y^{T}Y - 2W^{T}X^{T}Y + W^{T}X^{T}XW)$$

$$=\frac{\partial}{\partial \vec{W}} (Y^{T}Y) - \frac{\partial}{\partial \vec{W}} (2W^{T}X^{T}Y) + \frac{\partial}{\partial \vec{W}}(W^{T}X^{T}XW)$$

Taking our derivative of a linear term for the $$-2W^{T}X^{T}Y$$ term and derivative of a quadratic term for the $$W^{T}X^{T}XW$$ term, we can simplify into the following:

$$=0 -2X^{T}Y + 2X^{T}X$$

$$=-2X^{T}Y + 2X^{T}XW$$

We now minimize this derivative formulation by setting it equal to zero:

$$0 =-2X^{T}Y + 2X^{T}XW$$

$$= -X^{T}Y + X^{T}XW$$

$$X^{T}Y = X^{T}XW$$

$$(X^{T}X)^{-1}X^{T}Y = W$$

We've now proven that our solution for $$XW = y$$ also satisfies the least squares requirement of minimizing the residual sum of squares.

(see [here](https://pillowlab.princeton.edu/teaching/statneuro2018/slides/notes03b_LeastSquaresRegression.pdf) for more notes)
