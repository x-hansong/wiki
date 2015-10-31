---
title: Spring
layout: page
date: 2015-10-19
---
[TOC]

## Spring Data JPA映射表名大小写不敏感
> add at 2015-10-19

### 问题描述
使用`@Table(name="User")`指定表名的时候,表名会被映射为`user`,结果执行sql语句就会报找不到表名的错误.
### 解决办法
改变hibernate的naming-strategy

我使用的是spring boot,所以只需要在`application.properties`中加入

    spring.jpa.hibernate.naming-strategy=org.hibernate.cfg.DefaultNamingStrategy
### 问题原因
spring默认使用的hibernate naming-strategy是大小写不敏感的.
