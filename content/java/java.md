---
title: Java 实践
layout: page
date: 2016-02-28
---
[TOC]

## Java 数据类型

![](http://7xjtfr.com1.z0.glb.clouddn.com/20131118085507140.png)

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


## 动态代理
代理模式是常用的java设计模式，它的特征是代理类与委托类有同样的接口，代理类主要负责为委托类预处理消息、过滤消息、把消息转发给委托类，以及事后处理消息等。代理类与委托类之间通常会存在关联关系，一个代理类的对象与一个委托类的对象关联，代理类的对象本身并不真正实现服务，而是通过调用委托类的对象的相关方法，来提供特定的服务。

按照代理的创建时期，代理类可以分为两种：

1. 静态代理：由程序员创建或特定工具自动生成源代码，再对其编译。在程序运行前，代理类的.class文件就已经存在了。
2. 动态代理：在程序运行时，运用反射机制动态创建而成。

### JDK 动态代理
java.lang.reflect 包中的Proxy类和InvocationHandler 接口提供了生成动态代理类的能力。但是，JDK的动态代理依靠接口实现，如果有些类并没有实现接口，则不能使用JDK代理，这就要使用cglib动态代理了。

### Cglib动态代理
cglib是针对类来实现代理的，原理是用字节码技术，在运行时对指定的目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对final修饰的类进行代理。

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

## Determining Accessibility
- A package is always accessible.
- If a class or interface type is declared public, then it may be accessed by any code, provided that the compilation unit (§7.3) in which it is declared is observable.
If a top level class or interface type is not declared public, then it may be accessed only from within the package in which it is declared.
- An array type is accessible if and only if its element type is accessible.
- A member (class, interface, field, or method) of a reference (class, interface, or array) type or a constructor of a class type is accessible only if the type is accessible and the member or constructor is declared to permit access:

    - If the member or constructor is declared public, then access is permitted.All members of interfaces are implicitly public.
    - Otherwise, if the member or constructor is declared protected, then access is permitted only when one of the following is true:

        - Access to the member or constructor occurs from within the package containing the class in which the protected member or constructor is declared.
        - Access is correct as described in §6.6.2.

    - Otherwise, if the member or constructor is declared private, then access is permitted if and only if it occurs within the body of the top level class (§7.6) that encloses the declaration of the member or constructor.
    - Otherwise, we say there is default access, which is permitted only when the access occurs from within the package in which the type is declared.