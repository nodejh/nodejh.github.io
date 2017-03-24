+++
title = "Create and Manage MySQL User"
description = "创建 MySQL数据库及用户权限"
date = "2016-04-12T02:40:03+08:00"
tags = ["MySQL"]
categories = ["MySQL"]

+++


### 一.创建数据库用户

首先用root账号进入MySQL：

    $ mysql -u root -p

然后输入密码即可。

创建数据库并设置编码为 `utf8`，不然中文可能会乱码：

```
> create database mydb default character set utf8 defa
ult collate utf8_general_ci;
```

新建用户有两种方式

    1.> create user username@hostname identified by 'yourpassword';

    2.> insert into mysql.user(host,user,password) value('hostname','username',password('yourpassword'));
      > flush privileges;

<!--more-->

说明：
username：即将创建的用户名;
hostname：主机名。主机名为localhost表示用户可以本地登录；主机名为 % 表示远程访问，该用户可以从任意远程主机登陆。
yourpassword:该用户密码。如果密码为空，则该用户可不通过密码直接登录服务器。密码为空也可以直接写为：

     > create user 'username'@'hostname';

flsu privileges:刷新权限。
方法1设置后不需刷新权限即可生效；方法2需要刷新权限才能生效。

例：

     > create user test@localhost identified by 'testpasswd';
     > create user test@'%' identified by 'testpasswd';
创建一个用户名为test的用户，并设置其本地访问和远程访问。注意通配符 % 需要加引号。




### 二.查看已有用户
查看用户名，主机名，密码：

    > select user,host,password from mysql.user;


### 三.删除已有用户
删除也有两种方式。

    1.> drop user username@hostname;
    2.> delete from mysql.user where user='usrname' and host='hostname';

说明：如果未指定主机名，则 drop user username 默认删除username@'%'；如果没有 username@'%',则会提示 ERROR 1396 (HY000) 的错误。

### 四.设置修改用户密码
同样有两种方式。

    1.> set password for username@'hostname' = password('newpassword');
    2.> update mysql.user password=password('newpassword') where user='username' and host='hostname';
同样，方法一不需要刷新即可生效；方法2需要刷新才能生效。


### 五.为用户分配数据库及数据库表的权限

MySQL给用户分配数据库（表）权限的命令可概括为

    grant 权限 on 数据库对象 to 用户

##### 1.分配某个数据库的所有权限

    > grant all privileges on db.* to username@'%';
表示将数据库db中所有表的所有权限赋予用户username，通配符 % 表示可远程访问。

##### 2.分配数据库中一个表的所有权限

    > grant all privileges on db.table to username@'%';
表示将数据库db中的table表的所有权限赋予用户username。

##### 3.分配创建、修改、删除 MySQL 数据表结构权限。

    > grant create on db.* to username@'%';
    > grant alter  on db.* to username@'%';
    > grant drop   on db.* to username@'%';

  操作 MySQL 外键权限

    > grant references on db.* to username@'%';

  操作 MySQL 临时表权限

    > grant create temporary tables on db.* to username@'%';

  操作 MySQL 索引权限

    > grant index on db.* to username@'%';

  操作 MySQL 视图、查看视图源代码 权限

    > grant create view on db.* to username@'%';
    > grant show view on db.* to username@'%';

 操作 MySQL 存储过程、函数 权限

    > grant create routine on db.* to username@'%'; -- now, can show procedure status
    > grant alter routine on db.* to username@'%'; -- now, you can drop a procedure
    > grant execute on db.* to username@'%';


### 六.查看用户权限

查看当前用户权限

    > show grants;
查看其它用户权限

    > show grants for username@'%';


### 七.撤销已经赋予给 MySQL 用户权限的权限。

 revoke 与 grant 的语法类似，只需要把关键字 “to” 换成 “from” 即可：

    > grant all on *.* to username@'%';
    > revoke all on *.* from username@'%';


### 八.MySQL grant,revoke 用户权限注意事项

1. grant, revoke 用户权限后，该用户只有重新连接 MySQL 数据库，权限才能生效。

2. 如果想让授权的用户，也可以将这些权限 grant 给其他用户，需要选项 “grant option“

    > grant select on db.* to username@localhost with grant option;


这个特性一般用不到。实际中，数据库权限最好由 管理员 来统一管理。


---
Github Issue: [https://github.com/nodejh/nodejh.github.io/issues/7](https://github.com/nodejh/nodejh.github.io/issues/7)
