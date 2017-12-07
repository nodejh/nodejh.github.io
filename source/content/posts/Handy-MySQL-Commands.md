+++
categories = ["y"]
date = "2017-03-30T18:06:02+08:00"
description = "Handy MySQL Commands"
draft = false
tags = ["MySQL"]
title = "常用 MySQL 命令"

+++

<!--more-->


|描述|命令|
|:--|:--|
|登录远程数据库|mysql -h hostname -u root -p|
|创建数据库并设置编码|CREATE DATABASE mydatabase CHARACTER SET utf8 COLLATE utf8_general_ci|
|查看表结构|describe [table name]/desc table/show columns from [table name]|
|删除一列|alter table [table name] drop column [column name]|
|添加一列|alter table [table name] add column [new column name] varchar (20)|
|修改列名称|alter table [table name] change [old column name] [new column name] varchar (50)|
