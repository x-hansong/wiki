---
title: Java 集合类
layout: page
date: 2016-04-11
---
[TOC]

## HashMap
基于 JDK 1.8 分析
### put函数的实现
put函数大致的思路为：

1. 对key的hashCode()做hash，然后再计算index;
2. 如果没碰撞直接放到bucket里；
3. 如果碰撞了，以链表的形式存在buckets后；
4. 如果碰撞导致链表过长(大于等于TREEIFY_THRESHOLD)，就把链表转换成红黑树；
5. 如果节点已经存在就替换old value(保证key的唯一性)
6. 如果bucket满了(超过load factor*current capacity)，就要resize。

### get函数的实现
1. bucket里的第一个节点，直接命中；
2. 如果有冲突，则通过key.equals(k)去查找对应的entry
3. 若为树，则在树中通过key.equals(k)查找，O(logn)；
4. 若为链表，则在链表中通过key.equals(k)查找，O(n)。

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

## ConcurrentHashMap
ConcurrentHashMap 是一个并发散列映射表的实现，它允许完全并发的读取，并且支持给定数量的并发更新。相比于 HashTable 和用同步包装器包装的 HashMap（Collections.synchronizedMap(new HashMap())），ConcurrentHashMap 拥有更高的并发性。在 HashTable 和由同步包装器包装的 HashMap 中，使用一个全局的锁来同步不同线程间的并发访问。同一时间点，只能有一个线程持有锁，也就是说在同一时间点，只能有一个线程能访问容器。这虽然保证多线程间的安全并发访问，但同时也导致对容器的访问变成串行化的了。
在使用锁来协调多线程间并发访问的模式下，减小对锁的竞争可以有效提高并发性。有两种方式可以减小对锁的竞争：

1. 减小请求同一个锁的频率。
2. 减少持有锁的时间。

ConcurrentHashMap 的高并发性主要来自于三个方面：

1. 用分离锁实现多个线程间的更深层次的共享访问。
2. 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
3. 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。

使用分离锁，减小了请求同一个锁的频率。

通过 HashEntery 对象的不变性及对同一个 Volatile 变量的读 / 写来协调内存可见性，使得读操作大多数时候不需要加锁就能成功获取到需要的值(jdk6 中读到的值为空，则加锁重读，因为可能是其他线程在修改哈希表）。由于散列映射表在实际应用中大多数操作都是成功的读操作，所以 2 和 3 既可以减少请求同一个锁的频率，也可以有效减少持有锁的时间。

通过减小请求同一个锁的频率和尽量减少持有锁的时间 ，使得 ConcurrentHashMap 的并发性相对于 HashTable 和用同步包装器包装的 HashMap 有了质的提高。

### ConcurrentHashMap 的 get，clear 方法和迭代器是**弱一致性**的。

get 方法：没有加锁，如果在 get 的过程中，另一个线程写入或者删除某个元素，get 可能看不到修改后的结果。没有读锁是因为put/remove动作是个原子动作(比如put是一个对数组元素/Entry 指针的赋值操作)，读操作不会看到一个更新动作的中间状态。

clear 方法：因为没有全局的锁，在清除完一个segments之后，正在清理下一个segments的时候，已经清理segments可能又被加入了数据，因此clear返回的时候，ConcurrentHashMap中是可能存在数据的。因此，clear方法是弱一致的。

迭代器：在遍历过程中，如果已经遍历的数组上的内容变化了，迭代器不会抛出ConcurrentModificationException异常。如果未遍历的数组上的内容发生了变化，则有可能反映到迭代过程中。这就是ConcurrentHashMap迭代器弱一致的表现。

> 在 JDK 6,7,8 中 ConcurrentHashMap 的方法实现略有不同，但总体思想是一样的。

## LinkedHashMap
一个 Map 接口的实现，与 HashMap 不同的是，它维护了一个 Entry 的双向链表，因此其迭代的顺序是可预测的，支持按插入顺序或者访问顺序迭代元素。其 Entry 比 HashMap 的 Entry 多了首尾指针。

扩展HashMap增加双向链表的实现，号称是最占内存的数据结构。支持iterator()时按Entry的插入顺序来排序(但是更新不算， 如果设置accessOrder属性为true，则所有读写访问都算)。

实现上是在Entry上再增加属性before/after指针，插入时把自己加到Header Entry的前面去。如果所有读写访问都要排序，还要把前后Entry的before/after拼接起来以在链表中删除掉自己。

## ArrayList
以数组实现。节约空间，但数组有容量限制。超出限制时会增加50%容量，用System.arraycopy()复制到新的数组，因此最好能给出数组大小的预估值。默认第一次插入元素时创建大小为10的数组。

按数组下标访问元素--get(i)/set(i,e) 的性能很高，这是数组的基本优势。

直接在数组末尾加入元素--add(e)的性能也高，但如果按下标插入、删除元素--add(i,e), remove(i), remove(e)，则要用System.arraycopy()来移动部分受影响的元素，性能就变差了，这是基本劣势。

## LinkedList
以双向链表实现。链表无容量限制，但双向链表本身使用了更多空间，也需要额外的链表指针操作。

按下标访问元素--get(i)/set(i,e) 要悲剧的遍历链表将指针移动到位(如果i>数组大小的一半，会从末尾移起)。

插入、删除元素时修改前后节点的指针即可，但还是要遍历部分链表的指针才能移动到下标所指的位置，只有在链表两头的操作--add(), addFirst(),removeLast()或用iterator()上的remove()能省掉指针的移动。

## CopyOnWriteArrayList
并发优化的ArrayList。用CopyOnWrite策略，在修改时先复制一个快照来修改，改完再让内部指针指向新数组。

因为对快照的修改对读操作来说不可见，所以只有写锁没有读锁，加上复制的昂贵成本，典型的适合读多写少的场景。如果更新频率较高，或数组较大时，还是Collections.synchronizedList(list)，对所有操作用同一把锁来保证线程安全更好。

增加了addIfAbsent(e)方法，会遍历数组来检查元素是否已存在，性能可想像的不会太好。