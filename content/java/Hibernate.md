---
title: Hibernate 实践
layout: page
date: 2016-02-06
---
[TOC]

## Hibernate 关系映射
数据库表之间有两种关联方式，一种是三表关联，即通过一个中间表存储双方的ID键值对；第二种是双表关联，即一个为主表，从表存储主表的ID。

Hibernate 默认使用中间表关联，除非指定 mappedby 参数。
### mappedby 参数解释
mappedby 参数指定关系的拥有者，即主表。意味着外键在另一方。

使用注解的用法参考这篇文章，写的很详细。
[hibernate annotation注解方式来处理映射关系](http://www.cnblogs.com/xiaoluo501395377/p/3374955.html)