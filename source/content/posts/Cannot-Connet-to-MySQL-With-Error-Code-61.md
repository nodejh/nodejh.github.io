+++
title = "MySQL 连接错误 (61) "
description = "Cannot Connet to MySql With Error Code 61"
date = "2016-04-07T01:55:51+08:00"
tags = ["MySQL"]

+++

远程连接 MySQL 的时候出现如下错误:

```
Can't connect to MySQL server on '*.*.*.*' (61)
```
<!--more-->


这是因为 MySQL 默认的 `bind-address` 是 `127.0.0.1`, 也就是限制了只能本地访问.


所以解决方法之一,就是修改 `bind-address`.

改为 `0.0.0.0` ,任何远程 IP 都能访问.你也可以把它改为你的电脑的 IP,只能你自己访问.

修改后,重启就好了:

```
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf
# 修改 bind-address
$ sudo service mysql restart
```
---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/4](https://github.com/nodejh/nodejh.github.io/issues/4)
