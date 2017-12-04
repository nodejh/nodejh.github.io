+++
categories = ["y"]
date = "2017-02-06T22:30:26+08:00"
description = ""
draft = true
tags = ["x"]
title = "Schedule Tasks on Linux Using Crontab"

+++

通常我们需要执行一些计划任务，比如定期备份数据库、定时爬取某个网站的内容，这个时候就需要一个任务计划程序了。

<!--more-->


本文主要讲诉如何使用 crontab 来自定义计划任务，并给出几个周期性备份数据库的脚本示例。

## 1. cron、crontab

cron 就是一个在类 Unix 操作系统上的任务计划程序。它可以让用户在指定时间段周期性地运行命令或者 shell 脚本，通常被用在系统的自动化维护或者管理。

有很多 cron 的实现，crontab 是其中之一。crontab 本质上是一个定义计划任务的文件。

crond 进程启动之后，每分钟都会检测 crontab 文件是否有修改，如果有修改，则重新加载 crontab 文件，并执行其中的计划任务。所以编辑 crontab 文件之后，不需要重启 cron。

检查是否有要执行的任务。crontab 文件里面存储的则是用户定义的计划任务。

## 2. crontab 的使用

使用 `crontab -e` 命令可以编辑计划任务。第一次编辑的时候，会提示选择哪种编辑器进行编辑。crontab 计划任务的格式如下：

```
 ┌───────────── 分钟 (0 - 59)
 │ ┌───────────── 小时 (0 - 23)
 │ │ ┌───────────── 日 (1 - 31)
 │ │ │ ┌───────────── 月 (1 - 12)
 │ │ │ │ ┌───────────── 星期 (0 - 6) (0 表示星期天)
 │ │ │ │ │
 │ │ │ │ │
 │ │ │ │ │
 * * * * *  需要执行的命令
```

下面给出 crontab 一些示例，相信看了这些示例就能明白 crontab 怎么写了：

+

日志

通过邮件发送日志



[http://kvz.io/blog/2007/07/29/schedule-tasks-on-linux-using-crontab/](http://kvz.io/blog/2007/07/29/schedule-tasks-on-linux-using-crontab/)

[http://blog.csdn.net/zlzlei/article/details/7767599](http://blog.csdn.net/zlzlei/article/details/7767599)

[linux crontab命令参数及用法详解–linux自动化定时任务cron](http://blog.sae.sina.com.cn/archives/3572)


[crontab命令](http://man.linuxde.net/crontab)

[Linux 下执行定时任务 crontab 命令详解](https://segmentfault.com/a/1190000002628040)


[Linux Spool没有开启sendmail](http://blog.sina.com.cn/s/blog_6238358c01013cha.html)

[/var/spool/clientmqueue目录文件清理](http://yaksayoo.blog.51cto.com/510938/290069)

