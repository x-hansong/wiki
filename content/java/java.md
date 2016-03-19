---
title: Java 实践
layout: page
date: 2016-02-28
---
[TOC]

## Volatile
Java语言规范第三版中对volatile的定义如下： java编程语言允许线程访问共享变量，为了确保共享变量能被准确和一致的更新，线程应该确保通过排他锁单独获得这个变量。Java语言提供了volatile，在某些情况下比锁更加方便。如果一个字段被声明成volatile，java线程内存模型确保所有线程看到这个变量的值是一致的。

## JDK 排序算法
### JDK6
1. 针对基本类型，由于不要求排序是稳定的，使用快速排序

    1. 如果是小规模数组(size=7)，直接取中间元素作为枢轴
    2. 如果是中等规模数组(7<size<=40)，则在数组首、中、尾三个位置上的数中取中间大小的数作为枢轴
    3. 如果是大规模数组(size>40),则在9个指定的数中取一个伪中数(中间大小的数s)
2. 针对对象数组，为了保证排序是稳定的，JDK6采用了归并排序，同样也做了一些优化。

### JDK7
1. JDK7针对基本类型使用了Dual Pivot Quicksort,这种快速排序算法，相对于传统的单Pivot的快速排序效率要更好。
2. JDK7针对对象数组采用了TimSort, 这也是python的排序算法。

## String 类是 final 的
String objects are immutable, which means that once created, their values cannot be changed. The String class is not technically a primitive data type, but considering the special support given to it by the language, you'll probably tend to think of it as such.

## ConcurrentHashMap
ConcurrentHashMap 是一个并发散列映射表的实现，它允许完全并发的读取，并且支持给定数量的并发更新。相比于 HashTable 和用同步包装器包装的 HashMap（Collections.synchronizedMap(new HashMap())），ConcurrentHashMap 拥有更高的并发性。在 HashTable 和由同步包装器包装的 HashMap 中，使用一个全局的锁来同步不同线程间的并发访问。同一时间点，只能有一个线程持有锁，也就是说在同一时间点，只能有一个线程能访问容器。这虽然保证多线程间的安全并发访问，但同时也导致对容器的访问变成串行化的了。
在使用锁来协调多线程间并发访问的模式下，减小对锁的竞争可以有效提高并发性。有两种方式可以减小对锁的竞争：

1. 减小请求 同一个锁的 频率。
2. 减少持有锁的 时间。

ConcurrentHashMap 的高并发性主要来自于三个方面：

1. 用分离锁实现多个线程间的更深层次的共享访问。
2. 用 HashEntery 对象的不变性来降低执行读操作的线程在遍历链表期间对加锁的需求。
3. 通过对同一个 Volatile 变量的写 / 读访问，协调不同线程间读 / 写操作的内存可见性。

使用分离锁，减小了请求 同一个锁的频率。

通过 HashEntery 对象的不变性及对同一个 Volatile 变量的读 / 写来协调内存可见性，使得 读操作大多数时候不需要加锁就能成功获取到需要的值。由于散列映射表在实际应用中大多数操作都是成功的 读操作，所以 2 和 3 既可以减少请求同一个锁的频率，也可以有效减少持有锁的时间。

通过减小请求同一个锁的频率和尽量减少持有锁的时间 ，使得 ConcurrentHashMap 的并发性相对于 HashTable 和用同步包装器包装的 HashMap有了质的提高。

## final
### final data
1. 原生类型，final 使数据常量化
2. 使对象引用不变

final 允许声明变量时不初始化，但是编译器会保证该变量被使用前被初始化。

### final methods
1. final 函数无法在子类中被覆盖
2. 编译器可以将该函数内联，此时编译器不使用动态绑定
3. private 函数隐含 final

### final classes
无法被继承


## stactic
1. variable (also known as class variable)

    对于静态变量在内存中只有一个拷贝（节省内存），JVM只为静态分配一次内存，在加载类的过程中完成静态变量的内存分配，可用类名直接访问（方便），当然也可以通过对象来访问（但是这是不推荐的）。

    对于实例变量，没创建一个实例，就会为实例变量分配一次内存，实例变量可以在内存中有多个拷贝，互不影响（灵活）。

2. method (also known as class method)

    静态方法可以直接通过类名调用，任何的实例也都可以调用，

    因此静态方法中不能用this和super关键字，不能直接访问所属类的实例变量和实例方法(就是不带static的成员变量和成员成员方法)，只能访问所属类的静态成员变量和成员方法。

    类的构造方法隐含是 static。

3. block

    static代码块也叫静态代码块，是在类中独立于类成员的static语句块，可以有多个，位置可以随便放，它不在任何的方法体内，JVM加载类时会执行这些静态的代码块，如果static代码块有多个，JVM将按照它们在类中出现的先后顺序依次执行它们，每个代码块只会被执行一次。

4. nested class

    static修饰符使得嵌套类对象成为外部类的静态成员，与外部类直接关联。

    这样静态嵌套类作为一个静态成员，仅能访问外部类的静态成员，因为外部类中的非静态成员与外部类对象相关，静态嵌套类就不能访问他们，这使得静态嵌套类的功能变的很弱，可用之处很少。