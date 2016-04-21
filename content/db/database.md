---
title: 数据库基础知识
layout: page
date: 2016-04-09
---

[TOC]

## 数据库范式
1NF：字段是最小的的单元不可再分
2NF：满足1NF,表中的字段必须完全依赖于全部主键而非部分主键
3NF：满足2NF,非主键外的所有字段必须互不依赖

## 数据库事务
数据库事务(Database Transaction) ，是指作为单个逻辑工作单元执行的一系列操作。

### 事务的4个特性（ACID）：
1. 原子性（atomic）：事务必须是原子工作单元；对于其数据修改，要么全都执行，要么全都不执行。通常，与某个事务关联的操作具有共同的目标，并且是相互依赖的。原子性消除了系统处理操作子集的可能性。
2. 一致性（consistent）：事务在完成时，必须使所有的数据都保持一致状态。事务结束时，所有的内部数据结构（如B树索引或双向链表）都必须是正确的。
3. 隔离性（insulation）：由并发事务所作的修改必须与任何其它并发事务所作的修改隔离。事务查看数据时数据所处的状态，要么是另一并发事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看中间状态的数据。这称为可串行性，因为它能够重新装载起始数据，并且重播一系列事务，以使数据结束时的状态与原始事务执行的状态相同。当事务可序列化时将获得最高的隔离级别。在此级别上，从一组可并行执行的事务获得的结果与通过连续运行每个事务所获得的结果相同。由于高度隔离会限制可并行执行的事务数，所以一些应用程序降低隔离级别以换取更大的吞吐量。防止数据丢失。
4. 持久性（Duration）：事务完成之后，它对于系统的影响是永久性的。该修改即使出现致命的系统故障也将一直保持。


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