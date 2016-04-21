---
title: Java 集合类
layout: page
date: 2016-04-11
---
[TOC]

## HashMap

### put函数的实现
put函数大致的思路为：

1. 对key的hashCode()做hash，然后再计算index;
2. 如果没碰撞直接放到bucket里；
3. 如果碰撞了，以链表的形式存在buckets后；
4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
5. 如果节点已经存在就替换old value(保证key的唯一性)
6. 如果bucket满了(超过load factor*current capacity)，就要resize。

### get函数的实现
bucket里的第一个节点，直接命中；
如果有冲突，则通过key.equals(k)去查找对应的entry
若为树，则在树中通过key.equals(k)查找，O(logn)；
若为链表，则在链表中通过key.equals(k)查找，O(n)。

### hash函数的实现
在get和put的过程中，计算下标时，先对hashCode进行hash操作，然后再通过hash值进一步计算下标，如下图所示：

![](http://7xjtfr.com1.z0.glb.clouddn.com/293b52fc-d932-11e4-854d-cb47be67949a.png)

hash函数的实现：高16bit不变，低16bit和高16bit做了一个异或。

### resize的实现
当put时，如果发现目前的bucket占用程度已经超过了Load Factor所希望的比例，那么就会发生resize。在resize的过程，简单的说就是把bucket扩充为2倍，之后重新计算index，把节点再放到新的bucket中。

当超过限制的时候会resize，然而又因为我们使用的是2次幂的扩展(指长度扩为原来2倍)，所以，元素的位置要么是在原位置，要么是在原位置再移动2次幂的位置。

![](http://7xjtfr.com1.z0.glb.clouddn.com/ceb6e6ac-d93b-11e4-98e7-c5a5a07da8c4.png)

因此元素在重新计算hash之后，因为n变为2倍，那么n-1的mask范围在高位多1bit(红色)，因此新的index就会发生这样的变化：

![](http://7xjtfr.com1.z0.glb.clouddn.com/519be432-d93c-11e4-85bb-dff0a03af9d3.png)

因此，我们在扩充HashMap的时候，不需要重新计算hash，只需要看看原来的hash值新增的那个bit是1还是0就好了，是0的话索引没变，是1的话索引变成“原索引+oldCap”。

### HashMap的装箱空间效率
在笔试题中，一般“内存”是完全能够使用的，而在现实中HashMap空间效率之低，你却不一定知道。

比如定义了一个 HashMap<Long,Long>

1. Long的装箱

    在对象头中，加入额外的指针8Bype，加入8Bype的MarkWord(hashcode与锁信息)，这里就是16Byte。

    也就是说，long在装箱后，效率为 8/24 = 1/3

2. Map.Entry的装箱

    字段空间: hash(4) + padding(4) ＋ next(8) = 16Byte，这里的padding是字节对齐

    对象头: 16Byte，指针+MarkWord

    也就是说，维护一个Entry需要32Byte的空间

        static class Node<K,V> implements Map.Entry<K,V>
        {
            final int hash;
            final K key;
            V value;
            Node<K,V> next;
        }

3. 总效率

    8/(24 + 32) = 1/7


## LinkedHashMap
一个 Map 接口的实现，与 HashMap 不同的是，它维护了一个 Entry 的双向链表，因此其迭代的顺序是可预测的，支持按插入顺序或者访问顺序迭代元素。其 Entry 比 HashMap 的 Entry 多了首尾指针。

