---
title: Java 基础
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

## 数组
首先，数组类和普通的类是不同的。普通的类从class文件中加载，但是数组类由Java虚拟机在运行时生成。数组的类名是左方括号（[）+数组元素的类型描述符；数组的类型描述符就是类名本身。例如，int[]的类名是[I，int[][]的类名是[[I，Object[]的类名是[Ljava/lang/Object；，String[][]的类名是[[java/lang/String；，等等。

其次，创建数组的方式和创建普通对象的方式不同。普通对象由new指令创建，然后由构造函数初始化。基本类型数组由newarray指令创建；引用类型数组由anewarray指令创建；另外还有一个专门的multianewarray指令用于创建多维数组。

最后，很显然，数组和普通对象存放的数据也是不同的。普通对象中存放的是实例变量，通过putfield和getfield指令存取。数组对象中存放的则是数组元素，通过<t>aload和<t>astore系列指令按索引存取。其中<t>可以是a、b、c、d、f、i、l或者s，分别用于存取引用、byte、char、double、float、int、long或short类型的数组。另外，还有一个arraylength指令，用于获取数组长度。

## Serializable 接口的 serialVersionUID
The serialization runtime associates with each serializable class a version number, called a serialVersionUID, which is used during deserialization to verify that the sender and receiver of a serialized object have loaded classes for that object that are compatible with respect to serialization. If the receiver has loaded a class for the object that has a different serialVersionUID than that of the corresponding sender's class, then deserialization will result in an InvalidClassException. A serializable class can declare its own serialVersionUID explicitly by declaring a field named "serialVersionUID" that must be static, final, and of type long:

    ANY-ACCESS-MODIFIER static final long serialVersionUID = 42L;

If a serializable class does not explicitly declare a serialVersionUID, then the serialization runtime will calculate a default serialVersionUID value for that class based on various aspects of the class, as described in the Java(TM) Object Serialization Specification. However, it is strongly recommended that all serializable classes explicitly declare serialVersionUID values, since the default serialVersionUID computation is highly sensitive to class details that may vary depending on compiler implementations, and can thus result in unexpected InvalidClassExceptions during deserialization. Therefore, to guarantee a consistent serialVersionUID value across different java compiler implementations, a serializable class must declare an explicit serialVersionUID value. It is also strongly advised that explicit serialVersionUID declarations use the private modifier where possible, since such declarations apply only to the immediately declaring class--serialVersionUID fields are not useful as inherited members.

## Java 编码
Java 中的 String 类型（以及 char 类型）在内存中使用 UTF-16 编码。

由于 char 是 16 位固定长度，它的容量总是有限的，上限是 2**16=65536，能表示 0-65535（0x0000-0xFFFF）。即便满打满算它也只能表示 6 万多个不同字符而已，另一方面，Unicode 规划的字符空间高达 100 万以上，最新版本已经定义的字符也超过了 10 万。规划的码点范围具体为 U+0000 ~ U+10FFFF。char 能表示前面的 U+0000 ~ U+FFFF，对于 U+10000 ~ U+10FFFF 则无能为力。对于超过的 2 个字节的字符，需要两个 char 构成一个所谓的代理对才能表示一个抽象的字符。

参考[文本在内存中的编码(1)——乱码探源(4)](http://xiaogd.net/%E6%96%87%E6%9C%AC%E5%9C%A8%E5%86%85%E5%AD%98%E4%B8%AD%E7%9A%84%E7%BC%96%E7%A0%811-%E4%B9%B1%E7%A0%81%E6%8E%A2%E6%BA%904/)