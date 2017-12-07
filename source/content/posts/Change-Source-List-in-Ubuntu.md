+++
categories = ["Ubuntu"]
date = "2017-02-07T13:08:16+08:00"
description = "Change Source List in Ubuntu"
draft = false
tags = ["Ubuntu"]
title = "更改 Ubuntu 软件源"

+++

## 1. 软件管理工具 apt-get

Ubuntu 软件源本质上是一个软件仓库，我们可以通过 `sudo apt-get install <package-name>` 命令来从仓库中下载安装软件。

<!--more-->


上面命令中提到的 `apt-get` 则是 Ubuntu 系统中的一个包管理工具，其常用的几个命令如下：


**安装软件**

```
$ sudo apt-cache search <package-name>
```

**卸载软件**

```
$ sudo apt-get remove <package-name>
```

只卸载软件，不删除配置文件等。

**完全卸载**

```
$ sudo apt-get purge <package-name>
# 或
$ sudo apt-get remove <package-name> --purge
```

删除包括配置文件在内的所有文件。

**搜索软件**

```
$ sudo apt-cache <package-name>
```

更多的关于 `apt-get` 的用法可使用 `man apt-get` 命令查看。

## 2. 软件源的分类

Ubuntu 软件源分为两种：

+ Ubuntu 官方软件源
+ PPA 软件源

Ubuntu 官方软件源中包含了 Ubuntu 系统中所用到的绝大部分的软件，它对应的源列表是 `/etc/apt/sources.list`。

PPA 软件源即 Personal Package Archives（个人软件包档案）。有些软件没有被选入 UBuntu 官方软件仓库，为了方便Ubuntu用户使用，[Launchpad](https://launchpad.net) 提供了 PPA，允许用户建立自己的软件仓库，自由的上传软件。PPA也被用来对一些打算进入Ubuntu官方仓库的软件，或者某些软件的新版本进行测试。

Launchpad 是 Ubuntu 母公司 Canonical 有限公司所架设的网站，是一个提供维护、支援或联络 Ubuntu 开发者的平台。

## 3. 修改官方软件源

由于在国内从 Ubuntu 官方源下载软件比较慢，所以我们通常需要更换软件源来加快下载速度。互联网上有很多开源镜像站点，下面列出了一些网站。

选择源列表的时候，可以先使用 `ping` 命令测试一下网络速度，选择最快的源。

### 3.1 源列表

**Ubuntu Sources List Generator**

+ [Ubuntu Sources List Generator](https://repogen.simplylinux.ch)

可以在该网站上根据所处地区、Ubuntu发行版本等条件自动生成 Ubuntu 源。如我的 Ubuntu 版本号是 16.04，生成的源列表是：

```
#------------------------------------------------------------------------------#
#                            OFFICIAL UBUNTU REPOS                             #
#------------------------------------------------------------------------------#


###### Ubuntu Main Repos
deb http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial main restricted universe multiverse

###### Ubuntu Update Repos
deb http://cn.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://cn.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-security main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-updates main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb-src http://cn.archive.ubuntu.com/ubuntu/ xenial-backports main restricted universe multiverse
```

顺便给出查看 Ubuntu 版本号的命令：

```
$ lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 16.04 LTS
Release:	16.04
Codename:	xenial
```

**Ubuntu Wiki 源列表**

+ [Ubuntu Wiki 源列表](http://wiki.ubuntu.org.cn/源列表)

该站点提供了大量的开源镜像，可以先根据自己的版本号，点击版本号后面的 `[详细]` 链接进入 `模板: Ubuntu source` 页面，该页面提供了很多可用的服务器列表

### 3.2 修改源列表

1. 首先备份源列表(for sure)

    ```
    $ sudo cp /etc/apt/sources.list /etc/apt/sources.list_backup
    ```

2. 而后用gedit或其他编辑器打开

    ```
    $ sudo gedit /etc/apt/sources.list
    # 或 vim
    $ sudo vim /etc/apt/sources.list
    ```

3. 从上面的列表中选择合适的源，替换掉文件中所有的内容，保存编辑好的文件。**注意：一定要选对版**本
4. 然后，刷新列表

    ```
    $ sudo apt-get update
    ```

    **注意：一定要执行刷新**

## 4. 修改 PPA 源


添加 PPA 软件源的命令：

```
$ sudo add-apt-repository ppa:user/ppa-name
```

删除 PPA 软件源的命令：

```
$ sudo add-apt-repository --remove ppa:user/ppa-name
```

比如 FireFox PPA 源，[https://launchpad.net/~ubuntu-mozilla-daily/+archive/ppa](https://launchpad.net/~ubuntu-mozilla-daily/+archive/ppa) ，我们可以在这里找到 `ppa:ubuntu-mozilla-daily/ppa` 的字样，然后我们通过以下命令把这个源加入到 source list 中：

```
$ sudo apt-add-repository ppa:ubuntu-mozilla-daily/ppa
```

关于 PPA 的源，可以通过 Google 搜索，也可以在 [https://launchpad.net](https://launchpad.net) 网站搜索（可能效率比较低）。有的软件可能在官网提供了 PPA 源名称。

添加完成之后，就会在 `/etc/apt/sources.list.d/` 里面创建一个文件：`ubuntu-mozilla-daily-ubuntu-ppa-xenial.list`。现在来看看里面的内容：

```
$ cat /etc/apt/sources.list.d/ubuntu-mozilla-daily-ubuntu-ppa-xenial.list
deb http://ppa.launchpad.net/ubuntu-mozilla-daily/ppa/ubuntu xenial main
# deb-src http://ppa.launchpad.net/ubuntu-mozilla-daily/ppa/ubuntu xenial main
```

可以发现其源列表格式其实和官方源一模一样。之所以把 PPA 和官方源区分开来，是因为第三方源可能没有十足的保障。

添加完成之后，依旧需要使用 `sudo apt-get update` 来刷新。
