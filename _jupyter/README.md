# Jupyter notebooks

This directory contains Jupyter notebooks. These notebooks will be used for working on code snippets for specific blog posts.

Resources on how to convert Jupyter notebooks to pages: [here](https://www.kasimte.com/adding-and-including-jupyter-notebooks-as-jekyll-blog-posts), [here](https://www.linode.com/docs/guides/jupyter-notebook-on-jekyll/), and [here](https://cduvallet.github.io/posts/2018/03/ipython-notebooks-jekyll)
and [here](https://michaelwornow.net/2022/09/13/jupyter-notebook-to-markdown).

To set up the virtual environment using `conda`:

```{bash}
conda create --name .personal_website python=3.9
conda activate .personal_website
pip install -r requirements.txt
```

To compile the `requirements.in` file to generate the `requirements.txt` file:

```{bash}
pip install pip-tools
pip-compile requirements.in
```

Export each as a markdown:

```{bash}
jupyter nbconvert 2023-12-22-linear-regression-example.ipynb --to markdown
```

Move the markdown file to the `_posts` folder.

Add a header similar to the following to each markdown file:

```{bash}
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
```

Any images that are compiled should be moved over to be in the same path as the markdown file as well.