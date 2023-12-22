# Jupyter notebooks

This directory contains Jupyter notebooks. These notebooks will be used for working on code snippets for specific blog posts.

Resources on how to convert Jupyter notebooks to pages: [here](https://www.kasimte.com/adding-and-including-jupyter-notebooks-as-jekyll-blog-posts), [here](https://www.linode.com/docs/guides/jupyter-notebook-on-jekyll/), and [here](https://cduvallet.github.io/posts/2018/03/ipython-notebooks-jekyll)

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
