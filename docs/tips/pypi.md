---
layout: default
title: Python Project Management - Pypi Upload
nav_order: 3
parent: Tips
last_modified_date: 2024-10-03
---

# 0. Purpose

The purpose of this blog is to show just simplified version of how to start developing python project upload it to pypi.
When I googled, I found that there are many ways to to, but many of them are ambigious.
So, I'll try to write a detailed blog about it.

# 1. Project Structure

It's just a simple project structure.

```plaintext
.
├── My_Project 
│   ├── __init__.py
│   ├── dataclass.py
│   ├── main.py
│   └── util.py
├── README.md
├── pyproject.toml
├── sample
└── setup.py
```

In my own opinion, split main code and utility code is good to maintain the code.
And also, from python@3.7, we can use dataclass (like struct in C).
So, I recommend to use it for readability and accessibility.

## 1.1. Project Code Structure

```python
# My_Project/__init__.py
from .main import MyClass
```

for python packaging, we need to have `__init__.py` file in the directory.

```python
# My_Project/dataclass.py
from dataclasses import dataclass
from typing import List, Optional

@dataclass
class MyData:
    name: str
    age: int
    address: Optional[str] = None
    friends: List[str] = None
```

You can cusomize the dataclass as you want.

```python
# My_Project/main.py
from .util import MyUtil

class MyClass:
    def __init__(self):
        self.util = MyUtil()

    def print(self):
        print("MyClass")
        self.util.print()

if __name__ == "__main__":
    my_class = MyClass()
    my_class.print()
```

I usually use `main.py` to write main Class and test it.

```python
# My_Project/util.py
def print():
    print("test1234")

class MyUtil:
    def __init__(self):
        pass

    def print(self):
        print("MyUtil")
```

I just use `util.py` to write my custom classes and functions.
But, you can split it as you want functionally as you want.

## 1.2. `pyproject.toml` and `setup.py`

```toml
[project]
name = "My-Project"
version = "0.0.1"
description = "My Simple Testing Project"
readme = {file = "README.md", content-type = "text/markdown"}
dependencies = ["requests", "tqdm"]
maintainers = [{name = "h1ghl1kh7", email = "h1ghl1kh7@gmail.com"}]
license = {text = "MIT License"}

[tool.setuptools]
packages = ["My_Project"] # project folder name

[project.urls]
Repository = "https://github.com/h1ghl1kh7/My_Project"

[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

It is a simple `pyproject.toml` file, and when I uploaded my own project, I used this format.
It you want to know more about `pyproject.toml`, you can refer to [Python Packaging User Guide](https://packaging.python.org/en/latest/guides/writing-pyproject-toml/).

The content of `setup.py` is very simple.
It's because I use `pyproject.toml` to manage the project (You don't write anything in `setup.py` if you write `pyproject.toml`).

```python
# setup.py
from setuptools import setup

if __name__ == "__main__":
    setup()
```

# 2. Upload to Pypi

Register to [pypi.org](https://pypi.org/)
Get api from (Account Settings -> API Tokens)

When you upload the project, it asks you to input token every time.

So, I recommend to write token into `~/.pypirc` file.

```plaintext
[pypi]
  username = __token__
  password = pypi-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

If then, it automatically pass the token check process with this information.

```bash
python3 setup.py sdist bdist_wheel
python3 -m twine upload dist/*
```

Execute the above commands, package is built and uploaded.

It automatically create new project on pypi, so, you don't have to create project on site.

And, I recommend to remove `dist`, `build`, `My_Project.egg-info` directories and rebuild the project when you upload another version of the project.
