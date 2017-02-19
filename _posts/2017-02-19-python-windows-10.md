---
layout: single
title: "Setting up Python for data analytics on Windows 10"
category: Python
---

Nowadays, Python is known as one of the most popular programming languages in development as well as data analytics domains. However, the catch is that it is not very Windows-friendly as it is to Mac OS or Linux. Windows users can choose to offset the disadvantage by [installing Linux on Windows 10](https://www.lifewire.com/install-ubuntu-linux-windows-10-steps-2202108), or installing Python and its data analytics tools properly. This tutorial aims to smoothen the installation process as much as possible, and provides solutions to common technical issues that can happen during the setup.

# 1. Getting Anaconda and Python

Although we can always download Python from the [main website](https://www.python.org/downloads/), we can save so much time by getting [Anaconda](https://www.continuum.io/downloads) instead. Anaconda is a distribution platform for Python (as well as R), specialized for data science, so it comes with a lot of necessary tools and packages for data science such as `Jupyter` notebook, `pandas`, `numpy`, `matplotlib`, `scikit-learn` and so on. As Anaconda recommends, choose the Python version that you use the most as the default environment. If you are a totally newbie to Python, this [link](https://wiki.python.org/moin/Python2orPython3) would give you some insights to make a decision. Note that this decision is not final; we can always jump back and forth between different Python versions, which I will cover later in the post.

Install Anaconda to your computer, and you will have Python, Conda, and a lot of packages included. Remember to add Python into PATH (which is the default option in Anaconda). To be frank, unless we know what we should do, it's better to install Anaconda with its default setup rather than customizing it. To confirm the installation and the version, type the following command into the command prompt:

```
python -V
```
which outputs:
```
Python 3.6.0 :: Anaconda 4.3.0 (64-bit)
```
on my laptop.

# 2. Installing new packages via pip

To install new packages to Python, we can use `pip`. `pip` is the package management system used to distribute Python packages, which usually comes with Python in the installation process. If it's not, we can easily get it by the command:

```
python get-pip.py
```
Most of the time, we don't need to get pip but need to update it. We can do by the command:
```
python -m pip install -U pip
```
Now you're ready to download new packages! There are two ways to install packages via pip, automatically and manually.

### a. Automatically:

We install new packages by typing the following command into the command prompt:
```
pip install package-name
```
For example, if I want to install `plotly`, an interactive graphing library, I would use the command:
```
pip install plotly
```

### b. Manually:

Sometimes, for one reason or another, we would find that pip is unable to install a package automatically, and we would have to it manually. We can find most of the common Python packages [here](http://www.lfd.uci.edu/~gohlke/pythonlibs/). 

First, we need to check which version that is supported by our `pip`, by typing the command:

```
python
import pip
print(pip.pep425tags.get_supported())
```
Here is an example of output, from my laptop:

```
[('cp36', 'cp36m', 'win_amd64'), ('cp36', 'none', 'win_amd64'), ('py3', 'none', 'win_amd64'), ('cp36', 'none', 'any'), ('cp3', 'none', 'any'), ('py36', 'none', 'any'), ('py3', 'none', 'any'), ('py35', 'none', 'any'), ('py34', 'none', 'any'), ('py33', 'none', 'any'), ('py32', 'none', 'any'), ('py31', 'none', 'any'), ('py30', 'none', 'any')]
```

The output means that the compatible version contains `cp36` and `win_amd64`. Now if I want to install `xgboost`, I would need to get the file `xgboost‑0.6‑cp36‑cp36m‑win_amd64.whl`. 

After getting the file, we can either change directory to the folder that contains the file (my preference) or include the location of the file that we just downloaded, and install it manually via `pip`. For example, to install `xgboost`, I would type the following command:

```
pip install xgboost‑0.6‑cp36‑cp36m‑win_amd64.whl
```

`xgboost` should be sucessfully installed to the computer! We also can check the list of packages we have installed and their versions by:

```
pip list
```

