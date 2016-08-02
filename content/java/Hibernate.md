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

### 通过中间表的方式映射关系
Role 端的配置
```
    @OneToMany(fetch = FetchType.EAGER, cascade = CascadeType.ALL)
    @JoinTable(
            name = "role_privilege",
            joinColumns = @JoinColumn(name = "rid"),
            inverseJoinColumns = @JoinColumn(name = "pid")
    )
    private Set<Privilege> privileges;
```
Privilege 端无需做修改。

## 级联操作
- save-update: 级联保存(load以后如果子对象发生了更新,也会级联更新). 但它不会级联删除
- delete: 级联删除, 但不具备级联保存和更新
- all-delete-orphan: 在解除父子关系时,自动删除不属于父对象的子对象, 也支持级联删除和级联保存更新.
- all: 级联删除, 级联更新,但解除父子关系时不会自动删除子对象.
- delete-orphan:删除所有和当前对象解除关联关系的对象

注意：以上设在哪一段就是指对哪一端的操作而言，比如delete，如果设在one的一端的<set>属性里，就是当one被删除的时候，自动删除所有的子记录；

如果设在many一端的<many-to-one>标签里，就是在删除many一端的数据时，会试图删除one一端的数据，如果仍然有many外键引用one，就会报“存在子记录”的错误；如果在one的一端同时也设置了cascade＝“delete”属性，就会发生很危险的情况：删除many一端的一条记录，会试图级联删除对应的one端记录，因为one也设置了级联删除many，所以其他所有与one关联的many都会被删掉。

所以，千万谨慎在many一端设置cascade＝“delete”属性。故此cascade一般用在<one-to-one>和<one-to-many>中

此外，如果使用JPA的级联注解，需要配置`orphanRemoval=true`，才能支持级联删除

    @OneToMany(fetch = FetchType.EAGER,
            cascade = CascadeType.ALL,
            orphanRemoval = true
    )

    ╔═════════════╦═════════════════════╦═════════════════════╗
    ║   Action    ║  orphanRemoval=true ║   CascadeType.ALL   ║
    ╠═════════════╬═════════════════════╬═════════════════════╣
    ║   delete    ║     deletes parent  ║    deletes parent   ║
    ║   parent    ║     and orphans     ║    and orphans      ║
    ╠═════════════╬═════════════════════╬═════════════════════╣
    ║   change    ║                     ║                     ║
    ║  children   ║   deletes orphans   ║      nothing        ║
    ║    list     ║                     ║                     ║
    ╚═════════════╩═════════════════════╩═════════════════════╝