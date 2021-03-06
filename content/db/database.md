---
title: 数据库基础知识
layout: page
date: 2016-04-09
---

[TOC]

## 数据库范式
- 1NF：字段是最小的的单元不可再分
- 2NF：满足1NF,表中的字段必须完全依赖于全部主键而非部分主键
- 3NF：满足2NF,非主键外的所有字段必须互不依赖

通俗地理解三个范式，对于数据库设计大有好处。在数据库设计中，为了更好地应用三个范式，就必须通俗地理解三个范式(通俗地理解是够用的理解，并不是最科学最准确的理解)：

- 第一范式：1NF是对属性的原子性约束，要求属性(列)具有原子性，不可再分解；(只要是关系型数据库都满足1NF)
- 第二范式：2NF是对记录的惟一性约束，要求记录有惟一标识，即实体的惟一性；
- 第三范式：3NF是对字段冗余性的约束，它要求字段没有冗余。 没有冗余的数据库设计可以做到。

但是，没有冗余的数据库未必是最好的数据库，有时为了提高运行效率，就必须降低范式标准，适当保留冗余数据。具体做法是： 在概念数据模型设计时遵守第三范式，降低范式标准的工作放到物理数据模型设计时考虑。降低范式就是增加字段，允许冗余。

## 数据库事务
数据库事务(Database Transaction) ，是指作为单个逻辑工作单元执行的一系列操作。

### 事务的4个特性（ACID）：
1. 原子性（atomic）：事务必须是原子工作单元；对于其数据修改，要么全都执行，要么全都不执行。通常，与某个事务关联的操作具有共同的目标，并且是相互依赖的。原子性消除了系统处理操作子集的可能性。
2. 一致性（consistent）：事务在完成时，必须使所有的数据都保持一致状态。事务结束时，所有的内部数据结构（如B树索引或双向链表）都必须是正确的。
3. 隔离性（insulation）：由并发事务所作的修改必须与任何其它并发事务所作的修改隔离。事务查看数据时数据所处的状态，要么是另一并发事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看中间状态的数据。这称为可串行性，因为它能够重新装载起始数据，并且重播一系列事务，以使数据结束时的状态与原始事务执行的状态相同。当事务可序列化时将获得最高的隔离级别。在此级别上，从一组可并行执行的事务获得的结果与通过连续运行每个事务所获得的结果相同。由于高度隔离会限制可并行执行的事务数，所以一些应用程序降低隔离级别以换取更大的吞吐量。防止数据丢失。
4. 持久性（Duration）：事务完成之后，它对于系统的影响是永久性的。该修改即使出现致命的系统故障也将一直保持。

### 事务隔离级别

- Read Uncommitted（读未提交）

    未做控制，允许脏读、不可重复读、幻读。

- Read Committed（读已提交）

    不允许脏读，允许不可重复读、幻读。

- Repeatable Read（可重复读）

    不允许脏读、不可重复读、幻读，保证事务一致性（此为InnoDB默认隔离级别）。

- Serializable（串行化读）

    串行化读，通过锁来控制并发，读加读锁 (S锁)，写加写锁 (X锁)，读写间相互都会阻塞，所有事务都以串行执行，所以不存在脏读、不可重复读、幻读的问题。（该级别会导致并发度急剧下降，在MySQL/InnoDB下不建议使用）

注：

1. 根据SQL92标准，Repeatable Read并不能解决幻读问题，但MySQL/InnoDB通过Next-Key Lock机制在Repeatable Read下就可以避免幻读。
2. 并非所有数据库都提供SQL92标准的四种隔离级别，比如Oracle，只提供了SQL92标准中的Read Committed和Serializable，外加自己提供的Read-Only。
3. 不同数据库默认的事务隔离级别是不同的，比如Oracle，MSSQL，DB2默认是Read Committed。
4. Read Committed为语句级一致性读，Repeatable Read为事务级一致性读。

## SQL逻辑查询语句执行顺序
### SQL 语法
```
SELECT DISTINCT <select_list>
FROM <left_table>
<join_type> JOIN <right_table>
ON <join_condition>
WHERE <where_condition>
GROUP BY <group_by_list>
HAVING <having_condition>
ORDER BY <order_by_condition>
LIMIT <limit_number>
```

### SQL 执行顺序
```
(7)     SELECT
(8)     DISTINCT <select_list>
(1)     FROM <left_table>
(3)     <join_type> JOIN <right_table>
(2)     ON <join_condition>
(4)     WHERE <where_condition>
(5)     GROUP BY <group_by_list>
(6)     HAVING <having_condition>
(9)     ORDER BY <order_by_condition>
(10)    LIMIT <limit_number>
```
