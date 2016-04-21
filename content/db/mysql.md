---
title: MySQL
layout: page
date: 2015-10-31
---
[TOC]

## 基本命令
> add at 2015-10-31

### 运行sql脚本
    mysql> source my.sql;

### 登录mysql控制台
    mysql -h host -u username -p

## Mysql远程连接
> add at 2015-10-31

### 操作步骤
1. 登录mysql控制台
2. 修改mysql库中的user表,让某个用户拥有在任意机器上远程连接mysql的权限

        use mysql;
        //查看当前用户表中用户访问权限,user表的主键是host+user
        select host,user from user;
        //默认情况下,root的host有一个是`localhost`,代表用户只能在本机上连接mysql
        //我们可以换成`%`,代表用户可以在任意主机上连接mysql
        update user set host = '%' where user = 'root' and host = 'localhost';
        //刷新权限
        flush privileges;

3. 在其他机器上连接mysql

        mysql -h 'host ip' -u root -p

### 可能出现的问题
1. 在Ubuntu系统下可能会报错`ERROR 2003 (HY000): Can't connect to MySQL server on 'xxx.xxx.xxx.xxx' (111)`

    **解决方法:**

    - `vim /etc/mysql/my.cnf`
    - 注释掉`bind-address = 127.0.0.1`
    - 重启mysql

## SQL 语法
| 语法 | 语义 | 例子 |
| -  | - | - |
| create database 数据库名 [其他选项]; | 创建数据库 | create database samp_db character set gbk; |
| show databases; | 查看已经创建的数据库 |
| use 数据库名; | 选择某个数据库进行操作 |
| create table 表名称(列声明); | 创建表 | create table students（id int unsigned not null auto_increment primary key, name char(8) not null,sex char(4) not null, age tinyint unsigned not null, tel char(13) null default "-"); |
| show tables | 查看表 |
| describe 表名 | 查看表的结构 |
| show create table 表名 | 查看表的创建语句 |
| insert [into] 表名 [(列名1, 列名2, 列名3, ...)] values (值1, 值2, 值3, ...); | 插入值
| select 列名称 from 表名称 [查询条件]; | 查询表中的数据 |
| update 表名称 set 列名称=新值 where 更新条件; | 更新表中的数据
| delete from 表名称 where 删除条件; | 删除表中的数据
| alter table 表名 add 列名 列数据类型 [after 插入位置]; | 添加列
| alter table 表名 change 列名称 列新名称 新数据类型; | 修改列
| alter table 表名 drop 列名称; | 删除列
| alter table 表名 rename 新表名; | 重命名表
| drop table 表名; | 删除表
| drop database 数据库名; | 删除数据库


