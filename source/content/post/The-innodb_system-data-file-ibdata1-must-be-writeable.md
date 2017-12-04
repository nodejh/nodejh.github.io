+++
date = "2016-10-02T00:53:28+08:00"
description = "The innodb_system data file 'ibdata1' must be writeable"
title = "MySQL ibdata1 文件不可写"
tags = ["MySQL"]

+++

今天重启电脑后 MySQL 又用不了了！

然后查看了错误日志 ：

<!--more-->


```
$ sudo cat /usr/local/mysql/data/jh.local.err
2016-10-01T15:51:09.6NZ mysqld_safe Starting mysqld daemon with databases from /usr/local/mysql/data
2016-10-01T15:51:09.574413Z 0 [Warning] TIMESTAMP with implicit DEFAULT value is deprecated. Please use --explicit_defaults_for_timestamp server option (see documentation for more details).
2016-10-01T15:51:09.574540Z 0 [Warning] 'NO_ZERO_DATE', 'NO_ZERO_IN_DATE' and 'ERROR_FOR_DIVISION_BY_ZERO' sql modes should be used with strict mode. They will be merged with strict mode in a future release.
2016-10-01T15:51:09.574546Z 0 [Warning] 'NO_AUTO_CREATE_USER' sql mode was not set.
2016-10-01T15:51:09.574595Z 0 [Warning] Insecure configuration for --secure-file-priv: Current value does not restrict location of generated files. Consider setting it to a valid, non-empty path.
2016-10-01T15:51:09.574641Z 0 [Note] /usr/local/mysql/bin/mysqld (mysqld 5.7.13) starting as process 7326 ...
2016-10-01T15:51:09.579265Z 0 [Warning] Setting lower_case_table_names=2 because file system for /usr/local/mysql/data/ is case insensitive
2016-10-01T15:51:09.581901Z 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2016-10-01T15:51:09.581934Z 0 [Note] InnoDB: Uses event mutexes
2016-10-01T15:51:09.581943Z 0 [Note] InnoDB: GCC builtin __atomic_thread_fence() is used for memory barrier
2016-10-01T15:51:09.581950Z 0 [Note] InnoDB: Compressed tables use zlib 1.2.3
2016-10-01T15:51:09.582282Z 0 [Note] InnoDB: Number of pools: 1
2016-10-01T15:51:09.582394Z 0 [Note] InnoDB: Using CPU crc32 instructions
2016-10-01T15:51:09.583648Z 0 [Note] InnoDB: Initializing buffer pool, total size = 128M, instances = 1, chunk size = 128M
2016-10-01T15:51:09.594097Z 0 [Note] InnoDB: Completed initialization of buffer pool
2016-10-01T15:51:09.606687Z 0 [ERROR] InnoDB: The innodb_system data file 'ibdata1' must be writable
2016-10-01T15:51:09.606728Z 0 [ERROR] InnoDB: The innodb_system data file 'ibdata1' must be writable
2016-10-01T15:51:09.606753Z 0 [ERROR] InnoDB: Plugin initialization aborted with error Generic error
2016-10-01T15:51:09.913995Z 0 [ERROR] Plugin 'InnoDB' init function returned error.
2016-10-01T15:51:09.914027Z 0 [ERROR] Plugin 'InnoDB' registration as a STORAGE ENGINE failed.
2016-10-01T15:51:09.914035Z 0 [ERROR] Failed to initialize plugins.
2016-10-01T15:51:09.914040Z 0 [ERROR] Aborting

2016-10-01T15:51:09.914056Z 0 [Note] Binlog end
2016-10-01T15:51:09.914142Z 0 [Note] Shutting down plugin 'CSV'
2016-10-01T15:51:09.914491Z 0 [Note] /usr/local/mysql/bin/mysqld: Shutdown complete

```

其中最关键的当然是 `2016-10-01T15:51:09.606687Z 0 [ERROR] InnoDB: The innodb_system data file 'ibdata1' must be writable`。说的是 `ibdata1` 文件不可写。

`ibdata1` 是InnoDB的共有表空间，默认情况下会把表空间存放在一个文件ibdata1中，（此原因会造成这个文件越来越大）。

所以大概能猜测是 mysql 用户的权限不够了。所以再给 `ibdata1` 目录分配一下权限即可。

这个时候需要查看一下mysql 安装目录权限。我没有查看，就直接使用 `chown` 改变了整个 mysql 目录的权限，这是一个非常不好的习惯。

```
$ cd /usr/local/mysql
$ sudo chown -R _mysql:_mysql *
```

这里需要注意的是，macOS 系统下，mysql 的用户组和用户名都是 `_mysql`，Linux 没记错的话应该是 `mysql`。

然后再重启 MySQL：

```
$ sudo mysql.server start
Password:
Starting MySQL
 SUCCESS!
```

问题解决！

---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/1](https://github.com/nodejh/nodejh.github.io/issues/2)
