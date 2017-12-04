+++
categories = ["Database"]
date = "2017-02-17T15:03:43+08:00"
description = "Using SQL Plus on Oracle"
draft = false
tags = ["SQLPlus", "Oracle", "数据库"]
title = "使用 SQL *Plus 管理 Oracle 数据库"

+++

SQL *Plus 是基于命令行的 Oracle 管理工具，可以用来执行 `SQL`、`PL/SQL`、 和 `SQL*Plus` 命令：

<!--more-->


+ 支持查询、插入和更新数据
+ 执行 `PL/SQL` 程序
+ 查看表和对象的定义
+ 开发和执行批处理脚本
+ 进行数据库管理

## 登录 SQL *PLUS

直接登录，输入命令后会提示输入用户名密码：

```
$ sqlplus
```

使用用户名和密码：

```
$ sqlplus [username]/[user_password]
```

操作系统权限认证的 Oracle SYS 管理员登陆：

```
$ sqlplus / as sysdba
```

不在终端暴露密码登录：

```
$ sqlplus /nolog
SQL> conn [username]/[user_password]
# 或者
SQL > conn / as sysdba
```

退出登录：

```
SQL> exit
```

## 数据库信息

#### 查看数据库名

通常情况了我们称的 `数据库`，并不仅指物理的数据集合，而是物理数据、内存、操作系统进程的组合体。

```
SQL> select name from v$database;
```

#### 查询当前数据库实例名

> 实例是访问Oracle数据库所需的一部分计算机内存和辅助处理后台进程，是由进程和这些进程所使用的内存(SGA)所构成一个集合。

```
SQL> select instance_name from v$instance;
```

数据库实例名用于对外部连接。在操作系统中要取得与数据库的联系，必须使用数据库实例名。比如我们作开发，要连接数据库，就得连接数据库实例名，`orcl` 就为数据库实例名：

```
jdbc:oracle:thin:@localhost:1521:orcl
```

一个数据库可以有多个实例，在作数据库服务集群的时候可以用到。

## 用户管理

Oracle 使用 `PROFILE` 文件对用户访问资源的权限进行控制。

若不做特殊指定，创建用户时用户默认使用的 `PROFILE` 就是 `DEFAULT`。

查看当前用户：

```
SQL> show user
```

查看数据库用户：

```
SQL> select * from dba_users;
```

#### 解锁用户

默认当密码输错 10 次之后，用户就会被锁定：

```
ORA-28000: the account is locked
```

这个时候就需要管理员来解锁：

```
$ sqlplus / as sysdba
SQL> alter user [username] account unlock;
```

有解锁肯定就有锁定：

```
SQL> alter user [username] account lock;
```

#### 密码错误次数

当然，也可以自己修改最大密码错误次数，最大错误次数存储在 `dba_profiles` 表中。

首先根据 username 查看用户使用的 `PROFILE`：

```
SQL> SELECT PROFILE FROM DBA_USERS WHERE USERNAME='[username]'
```

然后根据 username 以及查询到的 `PROFILE` 查看该用户的最大密码错误次数 `FAILED_LOGIN_ATTEMPTS` ：

```
SQL> SELECT * FROM DBA_PROFILES WHERE PROFILE='DEFAULT' AND RESOURCE_NAME='FAILED_LOGIN_ATTEMPTS';
```

将错误次数修改为无限次：

```
SQL> ALTER PROFILE DEFAULT LIMIT FAILED_LOGIN_ATTEMPTSUNLIMITED;
```

#### 密码有效期

Oracle 11g 默认用户每三个月（180 天）就要修改一次密码，快到密码过期时间就会提醒：

```
ORA-28002: the password will expire within 7 days
```

这里同样要先查找到 `PROFILE` 再查看用户密码剩余过期时间：

```
SQL> SELECT * FROM DBA_PROFILES WHERE PROFILE='DEFAULT' AND RESOURCE_NAME='PASSWORD_LIFE_TIME';
```


修改密码有效期（不受限）：

```
SQL> ALTER PROFILE DEFAULT LIMIT PASSWORD_LIFE_TIMEUNLIMITED;
```

设置密码过期：

```
SQL> alter user [username] password expire;
```

#### 修改密码

修改当前登录用户密码：

```
SQL> password
```

修改某个用户的密码：

```
SQL> alter user [username] identified by [password];
```

## 表管理

Oracle 的表都是存储在表空间里面的。创建表之前需要先创建一个表空间。

#### 查看用户所拥有的表

查看用户所拥有的表：

```
SQL> SELECT TABLE_NAME FROM USER_TABLES;
```

查看用户可存取的表：

```
SQL> SELECT TABLE_NAME FROM ALL_TABLES;
```

数据库中所有表：

```
SQL> SELECT TABLE_NAME FROM DBA_TABLES;
```

#### 查看表空间

查看表空间详细数据文件：

```
SQL> SELECT FILE_NAME,TABLESPACE_NAME from DBA_DATA_FILES;
```

#### 创建表空间

```
create tablespace [表空间名称]
datafile [表空间数据文件路径 ]
size [表空间大小]
autoextend on;
```

例如：

```
SQL> create tablespace SoftwareManagement
  2  datafile '/data/oracle/oradata/orcl/SoftwareManagement.dbf'
  3  size 50m
  4  autoextend on;
```

#### 创建新用户

```
CREATE USER [用户名]
IDENTIFIED BY [密码]
DEFAULT TABLESPACE [表空间] (默认USERS)
TEMPORARY TABLESPACE [临时表空间] (默认TEMP)
```

例如：

```
SQL> create USER software
  2  identified by 123456
  3  default tablespace Softwaremanagement;
```

#### 分配权限

```
SQL> GRANT CONNECT TO [username];
SQL> GRANT RESOURCE TO [username];
SQL> GRANT DBA TO [username];  -- DBA为最高级权限，可以创建数据库、表等。
```

到这里，数据库中的表空间、用户以及用户权限都创建并分配好了，接下来用户就可以在自己的表空间中创建表，然后进行开发。


## 权限管理

在给用户分配权限的时候，分配了 `CONNECT`、`RESOURCE` 权限给用户。这两个权限到底是什么呢？

#### oracle中的权限

Oracle 中的权限分为两类：

+ 系统权限：系统规定用户使用数据库的权限，系统权限是对用户而言。
+ 实体权限：某种权限的用户对其他用户的表或视图的存取权限，是针对表或者视图而言。如 `select`、`update`、`insert`、`delete`、`alter`、`index`、`all`，其中 `all` 包含所有的实体权限。

#### 系统权限分类

+ DBA：拥有全部特权，是系统最高权限，只有DBA才可以创建数据库结构。
+ RESOURCE：拥有resource权限的用户只可以创建实体，不可以创建数据库结构。
+ CONNECT：拥有connect权限的用户只可以登录oracle，不可以创建实体，不可以创建数据库结构。

> 建议：
> 对于普通用户，授予 `CONNECT`、`RESOURCE` 权限；
> 对于 `DBA` 管理用户，授予 `CONNECT`、`RESOURCE`、`DBA` 权限。

## 导入导出

数据库的导入导出也是一个很常见的需求。

#### 导出

```
$ exp [username]/[password]@[orcl] file=./database.dmp  full=y
```

+ `username` 是数据库用户名
+ `password` 是数据库用户密码
+ `orcl` 是数据库实例名称
+ `file` 后面的参数是导出的数据库文件存放位置及文件名
+ `full` 其值为 `y` 表示全部导出，默认为 `no`。

如果只需导出某几张表，可以指定 `tables` 参数：`tables='(tableName, tableName1)'`。

#### 导入

```
$ imp [username]/[password]@[orcl] file=./database.dmp
```

和导出数据库语法一样，只是关键字不一样。

#### 执行 SQL 文件

执行 SQL 文件的方法有很多种。如下：

**使用 SQL PLUS 命令**

```
$ sqlplus [username]/password@[orcl] @path/file.name
```

或者远程执行：

```
$ sqlplus [username]/password@server_IP/service_name @path/file.name
```

如果sql脚本文件比较复杂，包含了begin end语句，就会不断显示行号，解决办法就是在 sql 脚本的最后用 `/` 符号结尾。

**在 SQL PLUS 中执行**

```
SQL>start file_path
```

```
SQL>@ file_path
```

其中 `file_path` 是文件路径。

---

参考

+ [Oracle表空间（tablespaces）](http://www.cnblogs.com/fnng/archive/2012/08/12/2634485.html)
+ [Oracle数据库，实例，表空间，用户，表之间的关系](http://zyjustin9.iteye.com/blog/2193804)
+ [Oracle 在Sqlplus 执行sql脚本文件](http://nvd11.blog.163.com/blog/static/2000183122012111524636835/)
