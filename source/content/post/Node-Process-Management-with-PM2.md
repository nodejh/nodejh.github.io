+++
categories = ["y"]
date = "2017-02-06T14:22:20+08:00"
description = "Node-Process-Management-with-PM2"
tags = ["x"]
title = "使用 PM2 管理 Node.js 进程"
draft = true

+++


## 查看 CPU 信息

linux

<!--more-->


```
# 查看物理CPU个数
$ cat /proc/cpuinfo| grep "physical id" | sort| uniq | wc -l

# 查看每个物理CPU中core的个数(即核数)
$ cat /proc/cpuinfo| grep "cpu cores"| uniq

# 查看逻辑CPU的个数
$ cat /proc/cpuinfo| grep "processor"| wc -l
```

macos

```
sysctl -n hw.ncpu
```

长期支持的版本号 奇数将在4月发布 偶数在10月



---

+ [pm2使用心得](http://www.jianshu.com/p/225b9284cfb8)
+ [Nodejs monitoring using PM2](https://codeforgeek.com/2016/02/nodejs-monitoring-using-pm2/)
+ [PM2 Doc](http://pm2.keymetrics.io/docs/usage/pm2-doc-single-page/)
+ [PM2实用入门指南](http://imweb.io/topic/57c8cbb27f226f687b365636)
+ [node.js使用cluster实现多进程](https://teakki.com/p/57dfa80a3c20b02e90a0d052)
+ [使用pm2管理Node.js应用](http://harttle.com/2016/09/07/pm2-express.html)
