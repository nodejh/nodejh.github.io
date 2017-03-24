+++
categories = ["python"]
date = "2017-02-10T16:14:03+08:00"
description = "pip 安装包出错: Could not import setuptools"
draft = true
tags = ["python", "pip"]
title = "Fix Could Not Import Setuptools from PIP"

+++

使用 pip 安装包的时候如 `pip install shadowsocks`，遇到了如下错误：

```
Could not import setuptools which is required to install from a source distribution.
Please install setuptools.
```

这是因为 python 环境中没有 setuptools 这个包。安装上就可以了：

```
$ pip install -U setuptools
```

更具体的内容可在 Github Issues 查看：

+ [ImportError: No module named setuptools (add details to docs) ](https://github.com/pypa/pip/issues/1064)

