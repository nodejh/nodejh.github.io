+++
categories = ["Database"]
date = "2017-03-26T00:42:23+08:00"
description = "Understand the Oracle startup process"
draft = false
tags = ["数据库", "Oracle", "Database"]
title = "深入理解 Oracle 启动原理"

+++

## 一. 常用启动步骤

对于普通用户，如果需要使用 Oracle 数据库，需要两个启动步骤：启动数据库和启动监听器。

<!--more-->


如果还需要使用 OEM 来监控数据库服务，则还要启动 OEM。

#### 1.1. 启动监听器

Oracle 监听器是一个独立的后台进程，用于监听客户端向数据库服务器端提出的连接请求，它是客户端和服务器端通讯的桥梁。



启动监听器：

```
$ lsnrctl start
```


#### 1.2. 启动数据库

我们可以使用 `sqlplus` 来启动数据库。关于 `sqlplus` 的详细使用方法请参考 [《使用 SQL *Plus 管理 Oracle 数据库》](https://github.com/nodejh/nodejh.github.io/issues/31)。

进入 `sqlplus` ：

```
$ sqlplus / as sysdba
```

启动数据库：

```
>SQL startup
```

完成这两个步骤，就可以使用数据库了。


#### 1.3. 启动 OEM

Oracle Enterprise Manager（Oracle企业管理器，简称 OEM ）是一个图形化数据库管理工具，可同时监控管理多个系统上的多个数据库，因而特别适合分布式环境。

启动 OEM：

```
$ emctl start dbconsole
```

启动成功后就可以通过 `http://服务器:1158/em` 来访问基于 Web 的监控页面。

## 二. 启动概述

Oracle Serve 由实例（Instance）和数据库（database）组成，每一个运行的 Oracle 数据库都与一个 Oracle 实例关联。

实例是由一组后台进程和一块称为系统全局区 SGA（System Global Area）的共享内存段组成。后台进程是数据库和操作系统进行交互的通道，后台进程的命名由 ORACLE_SID 决定，Oracle 根据 ORACLE_SID 来寻找参数文件启动实例。数据库是指存储在磁盘上的一组物理文件。

Oracle 数据库具有四种状态，启动过程具有三个阶段。

四种状态分别 `shutdown` `nomount` `mount` `open`，对应三个阶段分别为：

+ 启动实例 `shutdown --> nomount`
+ 装载数据库 `nomount --> mount`
+ 打开数据库 `mount --> open`

```
                                                    ^
                                           open     |
                                     +--------------|
                                     |   All files
                                     |   opened as
                                     | described by
                              mount  |  the control
                       +-------------+ file for this
                       | Control file    instance
                       |    opened
                       |   for this
               nomount |   instance
           +-----------+
           |  Instance
           |   started
           |
  shutdown |
-----------+
```

启动实例后，Oracle 软件会将实例与特定的数据库关联，这个过程称为装载数据库。接下来可以打开数据库，以便授权用户访问数据库。在同一台计算机上可以并发执行多个实例，每一个实例只访问自己的物理数据库。

## 三. 启动详解

为了弄清楚 Oracle 启动过程的详细内容，我们需要用到两个命令：

+ `ps` 用来查看系统运行了哪些进程
+ `ipcs` 查询进程间通信设施状态，显示使用了共享内存和信号量

#### 3.1. `shutdown` 状态

当 Oracle 处于该状态的时候，Oracle 的所有文件都静静的躺在磁盘里，一切都还未开始，属于关机状态。

```
$ ps -ef | grep oracle
oracle    4524     1  0 00:54 ?        00:00:39 /data/oracle/product/11.2.0/db_1/bin/emagent
root     12825   974  0 13:49 ?        00:00:00 sshd: oracle [priv]
oracle   12832 12825  0 13:49 ?        00:00:00 sshd: oracle@pts/0
oracle   12833 12832  0 13:49 pts/0    00:00:00 -bash
oracle   13825 12833  0 13:54 pts/0    00:00:00 ps -ef
oracle   13826 12833  0 13:54 pts/0    00:00:00 grep --color=auto oracle
$ ipcs -a

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x011268f0 458753     root       600        1000       8

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
```

#### 3.2. 启动实例 `shutdown --> nomount`


总体来说，启动数据库实例包括以下操作：

1. 读取参数文件 `SPFILE`
2. 分配 SGA
3. 启动后台进程
4. 打开告警文件和跟踪文件

在启动实例时，将为实例创建一系列后台进程和服务进程，并且在内存中创建 SGA 区等内存结构。在实例启动的过程中只会使用到初始化参数文件，数据库是否存在对实例的启动没有影响。如果初化参数设置有误，实例将无法启动。

启动数据库实例的命令如下：

```
$ sqlplus / as sysdba

SQL*Plus: Release 11.2.0.1.0 Production on Sun Mar 26 14:05:14 2017

Copyright (c) 1982, 2009, Oracle.  All rights reserved.

Connected to an idle instance.

SQL> startup nomount
ORACLE instance started.

Total System Global Area 3273641984 bytes
Fixed Size		    2217792 bytes
Variable Size		 2432698560 bytes
Database Buffers	  822083584 bytes
Redo Buffers		   16642048 bytes
```

启动数据库实例后，只会创建实例（即创建 Oracle 实例的各种内存结构与服务进程），并不加载数据库，也不会打开任何数据文件。

**测试数据文件能否打开：**

```sql
SQL> select * from v$datafile;
select * from v$datafile
              *
ERROR at line 1:
ORA-01507: database not mounted
```

`select * from v$datafile` 的时候报错，说明数据库文件在 `nomount` 状态下是无法访问的，因为数据字典需要从控制文件获取文件的信息，而此时控制文件没有打开所以无法查看。

但是在 `nomount` 状态下可以通过参数文件获得控制文件的位置，因为此时参数文件已经打开：

```sql
SQL> show parameter control_files;

NAME				     TYPE
------------------------------------
VALUE
------------------------------
control_files			     string
/data/oracle/oradata/orcl/cont
rol01.ctl, /data/oracle/flash_
recovery_area/orcl/control02.c
tl
```

###### 3.2.1 读取参数文件

启动数据库实例首先会读取 `SPFILE` 文件中的初始化参数，如果 `SPFILE` 文件不存在，则会读取初始化文件。Linux 系统的 `SPFILE` 文件在 `$ORACLE_HOME/dbs` 目录下，Windows NT 和 Windows 2000 中 `SPFILE` 文件目录在 `%ORACLE_HOME%\database`。文件的读取顺序如下：

+ `spfile$ORACLE_SID.ora`
+ `spfile.ora`
+ `init$ORACLE_SID.ora`

其中 `spfile$ORACLE_SID.ora` 和 `spfile.ora` 属于 `SPFILE` 文件，`init$ORACLE_SID.ora` 是初始化文件。

> 查看 `$ORACLE_HOME` 和 `$ORACLE_SID` 的值可使用 `echo` 命令，如 `echo $ORACLE_HOME`。

###### 3.2.2 分配 SGA

读取到参数文件之后，Oracle 会根据参数文件分配 SGA（System Global Area）。

SGA 是一个非常庞大的内存区间，这也是为什么开启 Oracle 之后占用了很大内存的原因。SGA 由所有服务进程和后台进程共享。

我们可以通过 `show sga` 或 `select * from v$sga` 查看 SGA 的大小：

```sql
SQL> show sga;
Total System Global Area 3273641984 bytes
Fixed Size		    2217792 bytes
Variable Size		 2432698560 bytes
Database Buffers	  822083584 bytes
Redo Buffers		   16642048 bytes

SQL> select * from v$sga;

NAME					      VALUE
---------------------------------------- ----------
Fixed Size				    2217792
Variable Size				 2432698560
Database Buffers			  822083584
Redo Buffers				   16642048

```

SGA 分为不同的池，我们可以通过视图 `v$sgastat` 查看：

```sql
SQL> select pool,sum(bytes) bytes from v$sgastat group by pool;

POOL			      BYTES
------------------------ ----------
java pool		   16777216
large pool		   16777216
shared pool		 1258295992
			  840943424

```


###### 3.2.3 启动后台进程

其中有 5 个进程必须启动， DBWR、LGWR、SMON、PMON、CKPT。

`SMON` 系统监视器（System Monitor）。如果 Oracle 实例失败，则在 SGA 中的任何没有写到磁盘中的数据都会丢失。有许多情况可能引起 Oracle 实例失败，例如操作系统的崩溃就会引起 Oracle 实例的失败。当实例失败之后，如果重新打开该数据库，则后台进程 SMON 自动执行实例的复原操作。

`DBWR` 数据库书写器（Database Write）。该服务器进程在缓冲存储区中记录所有的变化和数据。DBWR 把来自数据库的缓冲存储区中的脏数据写到数据文件中，以便确保数据库缓冲存储区中有足够的空闲的缓冲存储区。脏数据就是正在使用但是没有写到数据文件中的数据。

`LGWR` 日志书写器（Log Write）。LGWR 负责把重做日志缓冲存储区中的数据写入到重做日志文件中。

`CKPT` 检查点（Checkpoint）。该进程可以用来同步化数据库的文件，它可以把日志中的文件写入到数据库中。

`PMON` 进程监视器（Process Monitor）。当取消当前的事务，或释放进程占用的锁以及释放其它资源之后，PMON 进程清空那些失败的进程。


**查看系统进程和通信设施状态：**

```
$ ps -ef | grep oracle
oracle    4524     1  0 Mar26 ?        00:01:25 /data/oracle/product/11.2.0/db_1/bin/emagent
root      9797   974  0 00:57 ?        00:00:00 sshd: oracle [priv]
oracle    9805  9797  0 00:57 ?        00:00:00 sshd: oracle@pts/0
oracle    9806  9805  0 00:57 pts/0    00:00:00 -bash
oracle   10020     1  0 00:58 ?        00:00:00 ora_pmon_orcl
oracle   10022     1  0 00:58 ?        00:00:01 ora_vktm_orcl
oracle   10026     1  0 00:58 ?        00:00:00 ora_gen0_orcl
oracle   10028     1  0 00:58 ?        00:00:00 ora_diag_orcl
oracle   10030     1  0 00:58 ?        00:00:00 ora_dbrm_orcl
oracle   10032     1  0 00:58 ?        00:00:00 ora_psp0_orcl
oracle   10034     1  0 00:58 ?        00:00:01 ora_dia0_orcl
oracle   10036     1  0 00:58 ?        00:00:00 ora_mman_orcl
oracle   10038     1  0 00:58 ?        00:00:00 ora_dbw0_orcl
oracle   10040     1  0 00:58 ?        00:00:00 ora_lgwr_orcl
oracle   10042     1  0 00:58 ?        00:00:00 ora_ckpt_orcl
oracle   10044     1  0 00:58 ?        00:00:00 ora_smon_orcl
oracle   10046     1  0 00:58 ?        00:00:00 ora_reco_orcl
oracle   10048     1  0 00:58 ?        00:00:00 ora_mmon_orcl
oracle   10050     1  0 00:58 ?        00:00:00 ora_mmnl_orcl
oracle   10052     1  0 00:58 ?        00:00:00 ora_d000_orcl
oracle   10054     1  0 00:58 ?        00:00:00 ora_s000_orcl
oracle   15675  9806  0 01:59 pts/0    00:00:00 ps -ef
oracle   15676  9806  0 01:59 pts/0    00:00:00 grep --color=auto oracle
$ ipcs -a

------ Message Queues --------
key        msqid      owner      perms      used-bytes   messages

------ Shared Memory Segments --------
key        shmid      owner      perms      bytes      nattch     status
0x011268f0 458753     root       600        1000       9
0x00000000 524290     oracle     660        4096       0
0x00000000 557059     oracle     660        4096       0
0x2a1ee740 589828     oracle     660        4096       0

------ Semaphore Arrays --------
key        semid      owner      perms      nsems
0x19ba9600 1310738    oracle     660        154
```

由 `smon`、`pmon` `lgwr` 等进程可以看出此阶段创建了多个后台进程，并首次报告使用了共享内存和信号量。

###### 3.2.4 打开告警文件和跟踪文件

数据库的启动过程记录在警告追踪文件中，该警告追踪文件中包括数据库启动信息，它存放在参数`BACKGOUND_DUMP_DEST` 定义的目录下，警告日志的名字为 `alert_<sid>.log`，`sid` 是实例的名称：

```sql
SQL> show parameter background_dump_dest;

NAME			           TYPE               VALUE
-------------------- -------------------------------------------------
background_dump_dest  string  /data/oracle/diag/rdbms/orcl/orcl/trace
```

进入到目录查看警告日志关于 `startup nomount` 过程记录：

```
$ more alert_orcl.log
Fri Nov 11 17:04:51 2016
Starting ORACLE instance (normal)
LICENSE_MAX_SESSION = 0
LICENSE_SESSIONS_WARNING = 0
Shared memory segment for instance monitoring created
Picked latch-free SCN scheme 3
Using LOG_ARCHIVE_DEST_1 parameter default value as USE_DB_RECOVERY_FILE_DEST
Autotune of undo retention is turned on.
IMODE=BR
ILAT =27
LICENSE_MAX_USERS = 0
SYS auditing is disabled
Starting up:
Oracle Database 11g Enterprise Edition Release 11.2.0.1.0 - 64bit Production
With the Partitioning, OLAP, Data Mining and Real Application Testing options.
Using parameter settings in client-side pfile /data/oracle/admin/orcl/pfile/init.
```

Oracle 实例的后台进程会在遇到问题的时候将日志写入跟踪文件中。数据库的跟踪文件在目录由 `BACKGOUND_DUMP_DEST` 参数指定，最大大小由 `MAX_DUMP_FILE_SIZE` 指定，默认为`UNLIMITED`。

```sql
SQL> show parameter user_dump_dest;

NAME				     TYPE
------------------------------------ ----------------------
VALUE
------------------------------
user_dump_dest			     string
/data/oracle/diag/rdbms/orcl/o
rcl/trace
SQL> show parameter max_dump_file_size;

NAME				     TYPE
------------------------------------ ----------------------
VALUE
------------------------------
max_dump_file_size		     string
unlimited
```

#### 3.3 装载数据库 `nomunt --> mount`

###### 3.3.1 装载数据库概述

装载数据库就是把数据库文件和实例关联起来，包括以下三个步骤：

+ Oracle根据参数文件 `SPFILE` 中的参数找到控制文件
+ 打开控制文件
+ 从控制文件获得数据字典和重做日志文件的名字及位置

完成以上三步，没有任何错误的情况下，Oracle 就已经把实例和数据库关联起来了。

装载数据库有两种方式：

+ 一是直接启动数据库到 `mount` 状态：`startup mount`
+ 二是如果数据库已经启动到 `nomount` 状态，使用 `alter database mount` 把数据库切换到`mount` 状态

`alert database mount`：

```sql
SQL> alter database mount;

Database altered.
```

`startup mount`：

```
SQL> shutdown immediate;
ORA-01109: database not open


Database dismounted.
ORACLE instance shut down.
SQL> startup mount;
ORACLE instance started.

Total System Global Area 3273641984 bytes
Fixed Size		    2217792 bytes
Variable Size		 2432698560 bytes
Database Buffers	  822083584 bytes
Redo Buffers		   16642048 bytes
Database mounted.
```

###### 3.3.2 可以查询控制文件

这个时候我们就可以查询控制文件、数据文件和重做日志文件了。

```sql
SQL> select status from v$instance;

STATUS
------------------------
MOUNTED

SQL> select name from v$controlfile;

NAME
--------------------------------------------------------------------------------
/data/oracle/oradata/orcl/control01.ctl
/data/oracle/flash_recovery_area/orcl/control02.ctl

SQL> select name from v$datafile;

NAME
--------------------------------------------------------------------------------
/data/oracle/oradata/orcl/system01.dbf
/data/oracle/oradata/orcl/sysaux01.dbf
/data/oracle/oradata/orcl/undotbs01.dbf
/data/oracle/oradata/orcl/users01.dbf
/data/oracle/oradata/orcl/example01.dbf
/data/oracle/oradata/orcl/SoftwareManagement.dbf

6 rows selected.

SQL> select member from v$logfile;

MEMBER
--------------------------------------------------------------------------------
/data/oracle/oradata/orcl/redo03.log
/data/oracle/oradata/orcl/redo02.log
/data/oracle/oradata/orcl/redo01.log

```

###### 3.3.3 不能查询表、视图

但此时还不能查询数据库文件，如表和视图。所以对于普通用户而言，这个时候数据库还是不可用的。只有等到经历了最后一步 `打开数据库` 之后，才能使用数据库。

```sql
SQL> select * from tab;
select * from tab
              *
ERROR at line 1:
ORA-01219: database not open: queries allowed on fixed tables/views only

SQL> select * from scott.dept;
select * from scott.dept
                    *
ERROR at line 1:
ORA-01219: database not open: queries allowed on fixed tables/views only
```

#### 3.4 打开数据库 `mount --> open`

打开数据库时，实例将打开所有处于联机状态的数据文件和重做日志文件。

在此期间，Oracle 服务器将校验所有的数据文件和联机日志文件能否打开，并对数据库作一致性检查。

+ 如果出现一致性错误，`SMON` 进程将启动实例恢复
+ 如果任一数据文件或联机日志文件丢失，Oracle 服务器将报错

只有将数据库设置为打开状态后，数据库才处于正常状态，这时普通用户才能够访问数据库。

打开数据库也有两种方式。

**一是使用 `alter` 命令。**

```sql
>SQL alter database open;

Database altered.

SQL> select status from v$instance;

STATUS
------------------------
OPEN

```

**二是直接通过 `startup` 命令启动。**

`startup` 命令会逐步完成数据库启动的三个步骤（创建实例、装载数据库、打开数据库），将数据库启动到 `open` 状态：

```bash
SQL> startup
ORACLE instance started.

Total System Global Area 3273641984 bytes
Fixed Size		    2217792 bytes
Variable Size		 2432698560 bytes
Database Buffers	  822083584 bytes
Redo Buffers		   16642048 bytes
Database mounted.
Database opened.
```

启动之后，就可以访问数据文件了：

```sql
SQL> select * from scott.dept;

    DEPTNO DNAME			LOC
---------- ---------------------------- --------------------------
	10 ACCOUNTING			NEW YORK
	20 RESEARCH			DALLAS
	30 SALES			CHICAGO
	40 OPERATIONS			BOSTON
```

## 四. 关闭数据库

与启动数据库顺序相反，也分三个步骤：

+ `CLOSE`  关闭数据库（关闭数据文件）
+ `DISMOUNT` 卸载数据库（关闭控制文件）
+ `SHUTDOWN` 关闭 Oracle 实例

同时关闭模式也有多种。

#### 4.1. NORMAL

正常的关闭方式。如果对于关闭数据库的时间没有限制，通常采用这种方式。

以 `NORMAL` 方式关闭数据库，Oracle 将执行如下操作：

1. 阻止任何用户建立新的连接
2. 等待当前所有正在连接的用户主动断开连接
3. 当前所有用户的都断开连接后，将立即关闭数据库

#### 4.2. TRANSACTION

事务关闭方式，它的首要任务是保证当前所有活动的事务都可以被提交，并在尽可能短的时间内关闭数据库。

以事务方式关闭，Oracle将执行如下操作：

1. 阻止用户建立新连接和开始新事务
2. 等待所有活动事务提交后，再断开用户连接
3. 当所有活动事务提交完毕，用户断开连接后，关闭数据库

#### 4.3. IMMEDIATE

立即关闭方式，可以较快且安全的关闭数据库，是 DBA 经常采用的关闭数据库的方式。

立即关闭方式 Oracle执行如下操作：

1. 阻止用户建立新的连接和开始新的事务
2. 中断当前事务，回滚未提交事务
3. 强制断开所有用户连接和执行检查点把脏数据写到数据文件中
4. 关闭数据库

## 五. 为什么分为三步

现在问题来了，明明一步就可以把数据库启动起来，为什么 Oracle 要很麻烦地分为三步呢？

这样的步骤，对于普通用户来说是多余的，但对于 DBA 来说确实非常重要的。因为这三个步骤中，每一步都会启动对应的进程打开对应的文件，所以维护的时候，我们就可以准对具体的问题，在数据库的某种状态下进行数据库维护工作。

**nomount 状态**

`nomount` 状态不打开任何的控制文件及数据文件，所以我们可以在此阶段数据库创建、控制文件重建、特定的备份恢复等操作。

**mount 状态**

在加载完成以后，数据库还不可以被访问，所以在此阶段我们可以：

+ 重命名数据文件，移动数据文件位置等
+ 启用或关闭重做日志文件的归档及非归档模式
+ 实现数据库的完全恢复


## 六. 常见连接错误

#### 1. 未启动数据库实例

```
Connection Failed
ORA-12514: TNS:listener does not currently know of service requested in connect descriptor
```

#### 2. 未启动监听器

```
Oracle Connection Failed
ORA 12541: TNS:no listener
```


---

参考：

+ [Starting Up a Database](https://docs.oracle.com/cd/B28359_01/server.111/b28310/start001.htm#ADMIN10063)
+ [Oracle SGA详解](http://blog.itpub.net/9399028/viewspace-682015/)
+ [Oracle之内存结构（SGA、PGA）](http://blog.itpub.net/25264937/viewspace-694917/)
+ [Oracle 数据库启动过程](http://www.dongcoder.com/detail-441242.html)
+ [oracle数据库启动过程大揭秘 ](http://blog.itpub.net/29620680/viewspace-1153413/)
+ [Oracle学习交流（二）--oracle体系结构及启动流程](http://www.zhaokongnuan.com/2015/01/03/Oracle学习交流（二）-oracle体系结构及启动流程/)
+ [Oracle 数据库实例启动关闭过程](http://blog.csdn.net/leshami/article/details/5542983)
+ [Oracle学习笔记——数据库启动原理](http://www.jellythink.com/archives/1007)

---

> Github Issues [https://github.com/nodejh/nodejh.github.io/issues/35](https://github.com/nodejh/nodejh.github.io/issues/35)
