+++
title = "Troubleshoot MySQL Start Error"
description = "MySQL 启动报错问题排查"
date = "2016-07-31T17:05:44+08:00"
tags = ["MySQL"]

+++



今天使用 MySQL 的时候，莫名奇妙除了很多问题。在 Google 和 StackOverflow 搜索了一大堆，也没有找到很好解决办法。Anyway，最终机智的我还是把问题解决。

在此记录下整个排错过程。


#### 0. 系统环境

+ 操作系统 OS X EI Caption 10.11.6 (15G31)
+ MySQL 5.7.13
+ `/usr/local/mysql/bin` 和 `/usr/local/mysql/support-files` 都已经加入到了系统环境变量

<!--more-->

#### 1. 进入 MySQL 控制台报错

```
# 使用 mysql 命令进入 mysql 控制台
$ mysql -uroot -p
Enter password:
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
```

出现这个错误，第一时间想到的原因就是 MySQL 并没有启动。为了验证这个猜测，所以我们接下来看看 MySQL 的运行情况。

```
$ mysql.server status
/usr/local/mysql/support-files/mysql.server: line 365: pidof: command not found
 ERROR! MySQL is not running
```

可以发现， MySQL 没有运行。

这里使用的是 `mysql.server status` 命令来查看 MySQL 的运行状态。如果你没有将该命令加入到系统环境变量，则需要加上相应的路径才行，不然会有类似于 `zsh: command not found: mysql.server` 的错误，也就是说，没有这个命令。

如果没有将 MySQL 相应命令加入系统环境变量，一般需要这么来启动 MySQL：`/usr/local/mysql/support-files/mysql.server status`，进入 MySQL 控台则需要这样：`/usr/local/mysql/bin/mysql -uroot -p`。也就是在 `mysql.server` 或 `mysql` 等命令前加上路径。


#### 2. MySQL 启动报错

既然 MySQL 没有运行，那么我们肯定要启动 MySQL。

```
$ mysql.server start
Starting MySQL
. ERROR! The server quit without updating PID file (/usr/local/mysql/data/jh.local.pid).
$ mysql.server status
/usr/local/mysql/support-files/mysql.server: line 365: pidof: command not found
 ERROR! MySQL is not running
```

启动报错了。会不会是权限不够呢？于是我们再试试使用 `sudo` 来启动 MySQL：

```
Starting MySQL
...........................
```

`...` 一直在增长，MySQL 一直是正在启动，但就是无法启动成功。

#### 3. 排查 MySQL 启动报错原因

既然 MySQL 启动报错了，那么我们可以通过查看一些日志来推断 MySQL 的报错原因。
在 MySQL 的安装目录里面，有一个 `data` 目录 （`/usr/local/mysql/data`），这里吗有一个名为 `your_computer_name.local.err` 的文件（例如我的该文件名为 `jh.local.err` ），这个文件里面记录了 MySQL 启动的详细日志信息。

为了更准地得到我们想要的信息，我们可以先删掉改文件，然后再启动 MySQL。

```
$ sudo rm /usr/local/mysql/data/jh.local.err
$ sudo mysql.server start
Starting MySQL
..................
# 然后 Ctrl+C 停止进程
```

这个时候，再来看看 `jh.local.err` 里面的内容：

```
$ sudo tail /usr/local/mysql/data/jh.local.err
2016-07-31T15:31:35.414440Z 0 [ERROR] InnoDB: Unable to lock ./ibdata1 error: 35
2016-07-31T15:31:35.414531Z 0 [Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.
2016-07-31T15:31:36.417804Z 0 [ERROR] InnoDB: Unable to lock ./ibdata1 error: 35
2016-07-31T15:31:36.418940Z 0 [Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.
2016-07-31T15:31:37.422609Z 0 [ERROR] InnoDB: Unable to lock ./ibdata1 error: 35
2016-07-31T15:31:37.422654Z 0 [Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.
2016-07-31T15:31:38.427437Z 0 [ERROR] InnoDB: Unable to lock ./ibdata1 error: 35
2016-07-31T15:31:38.427670Z 0 [Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.
2016-07-31T15:31:39.431165Z 0 [ERROR] InnoDB: Unable to lock ./ibdata1 error: 35
2016-07-31T15:31:39.431205Z 0 [Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.

```

第一行就出现了 `[ERROR] InnoDB: Unable to lock ./ibdata1 error: 35`。然后第二行有个 `[Note] InnoDB: Check that you do not already have another mysqld process using the same InnoDB data or log files.`。

大概意思就是，启动 MySQL 的进程无法给 `ibdata1` 这个文件加锁。可能是因为有其他 `mysqld` 进程已经在使用该文件了。

然后后面一直重复这两行。所以我们大概可以猜测，当我们使用 `sudo mysql.server start` 这个命令来启动 MySQL 的时候，MySQL 一直在尝试启动，但由于进程占用，一直无法启动成功。

#### 4. 查看 mysqld 进程

既然可能是 `mysqld` 进程已经启动，那么就列出 `mysqld` 的进程看看：

```
$ ps xa | grep mysqld
8238   ??  Ss     0:03.74 /usr/local/mysql/bin/mysqld --user=_mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=/usr/local/mysql/data/mysqld.local.err --pid-file=/usr/local/mysql/data/mysqld.local.pid
10291 s003  R+     0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn mysqld
```

果然是有进程启动了，那么我们结束该进程，再重新启动 MySQL  试试：

```
$ sudo kill 8238
$ sudo mysql.server start
Starting MySQL
..................
```

然而还是和之前一样的情况。再看看进程启动情况？

```
10341   ??  Ss     0:00.38 /usr/local/mysql/bin/mysqld --user=_mysql --basedir=/usr/local/mysql --datadir=/usr/local/mysql/data --plugin-dir=/usr/local/mysql/lib/plugin --log-error=/usr/local/mysql/data/mysqld.local.err --pid-file=/usr/local/mysql/data/mysqld.local.pid
10343 s003  R+     0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn mysqld
```

`mysqld` 的进程依然存在，只是进程 ID 由之前的 `8238` 变成了 `10341`。多试几次，就会发现，当我们 kill 掉 `mysqld` 的进程后，它会自动重启。

#### 5. 查看进程占用文件

为了弄清楚已经启动的 `mysqld` 进程到底是为什么重启，我们可以先看看进程打开了哪些文件。

```
$ lsof -c mysqld
```

什么都没有。`lsof` 只会列出当前用户启动的进程打开的文件。那么加上 `sudo` 试试？

```
$ sudo lsof -c mysqld
COMMAND   PID   USER   FD     TYPE             DEVICE  SIZE/OFF     NODE NAME
mysqld  10634 _mysql  cwd      DIR                1,4       510 38686058 /usr/local/mysql-5.7.13-osx10.11-x86_64/data
mysqld  10634 _mysql  txt      REG                1,4  30794000 38685611 /usr/local/mysql-5.7.13-osx10.11-x86_64/bin/mysqld
mysqld  10634 _mysql  txt      REG                1,4    643792 38194604 /usr/lib/dyld
mysqld  10634 _mysql  txt      REG                1,4 558201098 39151946 /private/var/db/dyld/dyld_shared_cache_x86_64h
mysqld  10634 _mysql    0r     CHR                3,2       0t0      302 /dev/null
mysqld  10634 _mysql    1w     REG                1,4     71906 39198339 /usr/local/mysql-5.7.13-osx10.11-x86_64/data/mysqld.local.err
mysqld  10634 _mysql    2w     REG                1,4     71906 39198339 /usr/local/mysql-5.7.13-osx10.11-x86_64/data/mysqld.local.err
mysqld  10634 _mysql    3u  KQUEUE                                       count=0, state=0xa
.......
```

果然 `mysqld` 打开了很多文件！说明 `mysqld` 这个进程真实存在，并且占用了 MySQL 文件，而且是以 `root` 权限运行的。

#### 6. MySQL 到底为何而启动

继续推断，`root` 权限启动，并且自动重启，那估计和系统启动有关了。

在 Linux 系统下，`/etc/rc.*` 或者 `/etc/init` 目录下包含着系统里各个服务启动和停止的脚本，而 OS X 中则是由 `launchd` 做进程的管理和控制。

`launchd`  是OS X 从 10.4开始引入，用于用于初始化系统环境的关键进程，它是内核装载成功之后在OS环境下启动的第一个进程。采用这种方式配置启动进程，只需要一个 `plist` 文件。`/Library/LaunchDaemons` 目录下的 `plist` 文件都是系统启动后立即启动进程。使用 `launchctl` 命令加载/卸载 `plist` 文件，加载配置文件后，程序启动，卸载配置文件后程序关闭。

进入 `/Library/LaunchDaemons` 查看一下，确实是有一个和 MySQL 相关的 `plist` 文件 `com.oracle.oss.mysql.mysqld.plist`。

那么卸载该文件，然后再查看进程，验证是不是因为该 `plist`，所以 `mysqld` 才会不断重启：

```
$ sudo launchctl unload -w /Library/LaunchDaemons/com.oracle.oss.mysql.mysqld.plist
```

查看进程：

```
$ ps xa | grep mysqld
10924 s005  S+     0:00.00 grep --color=auto --exclude-dir=.bzr --exclude-dir=CVS --exclude-dir=.git --exclude-dir=.hg --exclude-dir=.svn mysqld
```

果然 `mysqld` 进程已经不存在了。说明推测是正确的，`mysqld` 之所以会以 `root` 权限启动，就是因为有一个启动 `mysqld` 的 `plist` 配置文件。

因为是修通了系统的进程启动管理配置，所以需要重启一下，`launchd` 才会在下次启动中生效。

重启后，再通过 `mysql.server`  来启动 MySQL，就会发现一切正常了！

```
$ sudo mysql.server start
Password:
Starting MySQL
. SUCCESS!
```

#### 7. 总结

加载 `plist` 配置的命令：

```
$ sudo launchctl load -w /Library/LaunchDaemons/com.oracle.oss.mysql.mysqld.plist
```

卸载 `plist` 配置的命令：

```
$ sudo launchctl unload -w /Library/LaunchDaemons/com.oracle.oss.mysql.mysqld.plist
```
所以折腾了那么久，其实就只是因为 OS X 特殊的启动管理机制造成的 `mysqld` 自动启动并占用进程，造成我们期望的 MySQL 启动方式没有正常运行。

卸载掉 `plsit` 配置并重启系统就好了！


---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/13](https://github.com/nodejh/nodejh.github.io/issues/13)
