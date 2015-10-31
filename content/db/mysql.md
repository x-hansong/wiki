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
