---
title: Java 并发
layout: page
date: 2015-03-22
---
[TOC]

## 线程的状态
1. 新建状态（New）：新创建了一个线程对象。
2. 就绪状态（Runnable）：线程对象创建后，其他线程调用了该对象的start()方法。该状态的线程位于可运行线程池中，变得可运行，等待获取CPU的使用权。
3. 运行状态（Running）：就绪状态的线程获取了CPU，执行程序代码。
4. 阻塞状态（Blocked）：阻塞状态是线程因为某种原因放弃CPU使用权，暂时停止运行。直到线程进入就绪状态，才有机会转到运行状态。阻塞的情况分三种：

    1. 等待阻塞：运行的线程执行wait()方法，JVM会把该线程放入等待池中。
    2. 同步阻塞：运行的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则JVM会把该线程放入锁池中。
    3. 其他阻塞：运行的线程执行sleep()或join()方法，或者发出了I/O请求时，JVM会把该线程置为阻塞状态。当sleep()状态超时、join()等待线程终止或者超时、或者I/O处理完毕时，线程重新转入就绪状态。

5. 死亡状态（Dead）：线程执行完了或者因异常退出了run()方法，该线程结束生命周期。

## Thread

### join(timeout) 方法
Waits at most millis milliseconds for this thread to die. A timeout of 0 means to wait forever.
This implementation uses a loop of this.wait calls conditioned on this.isAlive. As a thread terminates the this.notifyAll method is invoked. It is recommended that applications not use wait, notify, or notifyAll on Thread instances

## Java 对象头与锁

Java对象有包含两个Word的头部。其中，第一个word被称为mark word，用来包含和垃圾收集、hash码的一些信息。第二个word用来指向对象的类。

![](http://7xjtfr.com1.z0.glb.clouddn.com/051358589904735.jpg)

第一个word的解释比较复杂，它包含两个bit用于指明mark word的用途。

- 00: 表明这个对象处于lightweight locked的状态，其余的bit表示一个指向lock record的指针
- 01:

    - 001: 表明这个对象没有被锁住，其余的bit表示hash, age
    - 101: 表明这个对象可以被biasable的锁住，其余的bit标明biasable锁的threadID, epoch和age.

- 10: 表明这个对象被锁住，并且有冲突发生，因而正在被复杂而低效的操作系统同步机制保护。剩余的bit是一个指向monitor的指针。
- 11: 这个对象被GC标明为垃圾

接下来，我们要谈谈对锁的优化。从文章《HotSpot: A new breed of virtual machine
》中，我们可以看到早期的JVM花在线程同步上的开销是惊人的，线程同步的开销占到了执行时间的21%。而且这个开销不能简单的被JIT优化掉。因此，众多牛人提出了对锁的很多优化。

目前，JDK中最新的Biased锁是基于下面的观察：

1. 轻量级锁处理中必定会用到的CAS(Compare and Set)操作相当慢，需要超过百个以上的cpu cycle
2. 通常一个锁只被一个线程访问。而且可能是被同一个线程反复的访问，因此反复的CAS会很大程度上影响应用程序的性能
Biased锁将会尝试声明某个对象属于一个特定的线程，这个线程对对象的lock和unlock操作不需要进行CAS。如果其他线程想lock这个对象，那么它会把这个线程的biased的状态撤销，使其利用普通的锁机制。这样的话，在JDK中lock和unlock的流程就是这样的：

        void lock(Object obj, Thread currentTr){
            if( obj biased to currentTr)
                return;
            if( obj biased to other thread)
                pause owner thread at safe point
                change mark word and lock record to pretend that obj is locked by other thread with general lock.
            else{
                 //fall to common lock
            }
        }

        void unlock(Object obj, Thread currentTr){
            if( obj biased to currentTr)
                return .
            else
                fall to common lock
        }

## JVM中锁的优化
简单来说在JVM中monitorenter和monitorexit字节码依赖于底层的操作系统的Mutex Lock来实现的，但是由于使用Mutex Lock需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的；然而在现实中的大部分情况下，同步方法是运行在单线程环境（无锁竞争环境）如果每次都调用Mutex Lock那么将严重的影响程序的性能。不过在jdk1.6中对锁的实现引入了大量的优化，如锁粗化（Lock Coarsening）、锁消除（Lock Elimination）、轻量级锁（Lightweight Locking）、偏向锁（Biased Locking）、适应性自旋（Adaptive Spinning）等技术来减少锁操作的开销。

- 锁粗化（Lock Coarsening）：也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。
- 锁消除（Lock Elimination）：通过运行时JIT编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本地Stack上进行对象空间的分配（同时还可以减少Heap上的垃圾收集开销）。
- 轻量级锁（Lightweight Locking）：这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态（即单线程执行环境），在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在monitorenter和monitorexit中只需要依靠一条CAS原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒（具体处理步骤下面详细讨论）。
- 偏向锁（Biased Locking）：是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令，因为CAS原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟（可参考这篇文章）。
- 适应性自旋（Adaptive Spinning）：当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁（mutex semaphore）前会进入忙等待（Spinning）然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore（即互斥锁）进入到阻塞状态。

## Unsafe 与 CAS

### Unsafe
简单讲一下这个类。Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门，JDK中有一个类Unsafe，它提供了硬件级别的原子操作。

这个类尽管里面的方法都是public的，但是开发者是无法使用它的，JDK API文档也没有提供关于这个类的方法解析。对于Unsafe类的使用都是受限制的，只有授信的代码才能获得该类的实例，当然JDK库里面的类是可以随意使用的。

从第一行的描述可以了解到Unsafe提供了硬件级别的操作，比如说获取某个属性在内存中的位置，比如说修改对象的字段值，即使它是私有的。不过Java本身就是为了屏蔽底层的差异，对于一般的开发而言也很少会有这样的需求。

例如下面的方法，分别用来分配内存，扩充内存和释放内存的：

    public native long allocateMemory(long paramLong);
    public native long reallocateMemory(long paramLong1, long paramLong2);
    public native void freeMemory(long paramLong);

### CAS
CAS，Compare and Swap即比较并替换，设计并发算法时常用到的一种技术，java.util.concurrent包全完建立在CAS之上，没有CAS也就没有此包，可见CAS的重要性。

当前的处理器基本都支持CAS，只不过不同的厂家的实现不一样罢了。CAS有三个操作数：内存值V、旧的预期值A、要修改的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做并返回false。

CAS也是通过Unsafe实现的，看下Unsafe下的三个方法：

    public final native boolean compareAndSwapObject(Object paramObject1, long paramLong, Object paramObject2, Object paramObject3);
    public final native boolean compareAndSwapInt(Object paramObject, long paramLong, int paramInt1, int paramInt2);
    public final native boolean compareAndSwapLong(Object paramObject, long paramLong1, long paramLong2, long paramLong3);

下面是 AtomicInteger 中 `addAndGet()`方法的定义：

    public final int addAndGet(int delta) {
        for (;;) {
            int current = get();
            int next = current + delta;
            if (compareAndSet(current, next))
                return next;
        }
    }

这段代码如何在不加锁的情况下通过CAS实现线程安全，我们不妨考虑一下方法的执行：

1. AtomicInteger里面的value原始值为3，即主内存中AtomicInteger的value为3，根据Java内存模型，线程1和线程2各自持有一份value的副本，值为3
2. 线程1运行到第三行获取到当前的value为3，线程切换
3. 线程2开始运行，获取到value为3，利用CAS对比内存中的值也为3，比较成功，修改内存，此时内存中的value改变比方说是4，线程切换
4. 线程1恢复运行，利用CAS比较发现自己的value为3，内存中的value为4，得到一个重要的结论-->此时value正在被另外一个线程修改，所以我不能去修改它
5. 线程1的compareAndSet失败，循环判断，因为value是volatile修饰的，所以它具备可见性的特性，线程2对于value的改变能被线程1看到，只要线程1发现当前获取的value是4，内存中的value也是4，说明线程2对于value的修改已经完毕并且线程1可以尝试去修改它
6. 最后说一点，比如说此时线程3也准备修改value了，没关系，因为比较-交换是一个原子操作不可被打断，线程3修改了value，线程1进行compareAndSet的时候必然返回的false，这样线程1会继续循环去获取最新的value并进行compareAndSet，直至获取的value和内存中的value一致为止

整个过程中，利用CAS保证了对于value的修改的线程安全性

### CAS 的缺点
CAS看起来很美，但这种操作显然无法涵盖并发下的所有场景，并且CAS从语义上来说也不是完美的，存在这样一个逻辑漏洞：如果一个变量V初次读取的时候是A值，并且在准备赋值的时候检查到它仍然是A值，那我们就能说明它的值没有被其他线程修改过了吗？如果在这段期间它的值曾经被改成了B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。这个漏洞称为CAS操作的"ABA"问题。java.util.concurrent包为了解决这个问题，提供了一个带有标记的原子引用类"AtomicStampedReference"，它可以通过控制变量值的版本来保证CAS的正确性。不过目前来说这个类比较"鸡肋"，大部分情况下ABA问题并不会影响程序并发的正确性，如果需要解决ABA问题，使用传统的互斥同步可能回避原子类更加高效。

*参考文章*
[Unsafe 与 CAS](http://www.cnblogs.com/xrq730/p/4976007.html)
