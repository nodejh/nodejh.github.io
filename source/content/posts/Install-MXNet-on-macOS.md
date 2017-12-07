+++
date = "2016-10-27T15:37:29+08:00"
description = "Build MXNet on macOS"
title = "在 macOS 中编译安装 MXNet"
tags = ["MXNet", "Deep Learning"]

+++

MXNet 是一个深度学习系统。关于 MXNet 的介绍可以看这篇文章：[《MXNet设计和实现简介》](https://github.com/dmlc/mxnet/issues/797)。

<!--more-->

在 macOS 上编译安装 MXNet 的大体步骤都是按照官方文档来进行安装即可。但由于每个人电脑环境不同，所以可能会出现一些依赖库／包的缺失，导致安装失败。

#### 安装依赖软件

在 macOS 上，首先需要具有以下软件：

+ Homebrew (to install dependencies)
+ Git (to pull code from GitHub)
+ Homebrew/science (for linear algebraic operations)
+ OpenCV (for computer vision operations)

如果上述已经安装了，就不需要再安装；如果没有，则按照下面的步骤安装：

```
# 安装 Homebrew
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# 安装 Git 和 OpenCV
$ brew update
$ brew install git
$ brew tap homebrew/science
$ brew info opencv
$ brew install opencv
```

#### 编译 MXNet

```
# 下载源码
$ git clone --recursive https://github.com/dmlc/mxnet
```

然后还需要安装 `openblas`：

```
# 安装 openblas
$ brew install --fresh -vd openblas
...
Generally there are no consequences of this for you. If you build your
own software and it requires this formula, you'll need to add to your
build variables:

    LDFLAGS:  -L/usr/local/opt/openblas/lib
    CPPFLAGS: -I/usr/local/opt/openblas/include

==> Summary
🍺  /usr/local/Cellar/openblas/0.2.18_2: 20 files, 41.8M, built in 12 minutes 33 seconds
```

如果没有安装 `openblas`，则会有类似 `fatal error: 'cblas.h' file not found` 的错误，详见 [https://github.com/dmlc/mxnet/issues/572](https://github.com/dmlc/mxnet/issues/572)。

接下来修改配置文件：

```
$ cd mxnet
$ cp make/osx.mk ./config.mk
```

用 vim 或其他编辑器打开 `config.mk`，在 `USE_BLAS = apple` 下面加入如下 `ADD_LDFLAGS = -I/usr/local/opt/openblas/lib` 和 `ADD_CFLAGS =  -I/usr/local/opt/openblas/include`：

```
USE_BLAS = apple
ADD_LDFLAGS = -I/usr/local/opt/openblas/lib
ADD_CFLAGS =  -I/usr/local/opt/openblas/include
```

最后再编译即可：

```
$ make -j$(sysctl -n hw.ncpu)
```

#### 在 Python 中使用 MXNet

编译安装完成之后，若要使用 MXNet 的 Python 接口，还需要将 mxnet/python 添加到 Python 的包搜索路径。至少有三种方式可以实现。

**1. python 代码手动加载**

```
import os, sys;
cur_path = os.path.abspath(os.path.dirname(__file__));
mxnet_lib_path = os.path.join(cur_path, 'mxnet/python');
sys.path.append(mxnet_lib_path);
import mxnet as mx;
```

在没有将 `mxnet/python` 添加到 PYTHONPATH 之前，依旧可以运行 `/example/image-classification` 里面的一些测试案例，就是因为案例里面有一行 `import find_mxnet`，而 `find_mxnet` 的作用就是手动加载 `mxnet/python`：

```
# find_mxnet.py
try:
    import mxnet as mx
except ImportError:
    import os, sys
    curr_path = os.path.abspath(os.path.dirname(__file__))
    sys.path.append(os.path.join(curr_path, "../../python"))
    import mxnet as mx

```

**2. 将路径加到环境变量 PYTHONPATH 中**

这种方法需要修改 shell 的配置文件。如果使用的 bash，则修改 `~/.bashrc`；若使用的是 zsh，则修改 `~/.zshrc`；其他类似。

在 bash 配置文件中加入下面这一行：

```
export PYTHONPATH=path_to_mxnet_root/python
```

其中 `path_to_mxnet_root` 是下载的 mxnet 源码目录。

**3. 全局安装 mxnet**

直接运行 `mxnet/python/setup.py`，将 mxnet 添加到全局路径即可：

```
python setup.py install --user
```

运行上面的命令后，脚本会在 `~/.local` 目录下创建一个 `lib` 目录，里面有一个 `python-2.7/site-packages` 文件夹。

如果是 `sudo python setup.py install`，则上面的目录会在 `/usr/lib` 下。


---

+ [Build MXNet on OS X (Mac)](http://mxnet.io/get_started/setup.html#build-mxnet-on-os-x-mac)
+ [mxnet的python包导入的前前后后](http://www.cnblogs.com/dengdan890730/p/5587542.html)


---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/10](https://github.com/nodejh/nodejh.github.io/issues/10)
