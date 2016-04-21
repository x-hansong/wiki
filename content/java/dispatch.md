---
title: Java 分派（重载与重写）
layout: page
date: 2016-04-20
---
[TOC]

## 重载与重写
方法重载（overload）：

1. 必须是同一个类
2. 方法名（也可以叫函数）一样
3. 参数类型不一样或参数数量不一样

方法的重写（override）两同两小一大原则：

1. 方法名相同，参数类型相同
2. 子类返回类型小于等于父类方法返回类型，
3. 子类抛出异常小于等于父类方法抛出异常，
4. 子类访问权限大于等于父类方法访问权限。

## 静态分派

    public class StaticDispatch {
        static abstract class Human{

        }

        static class Man extends Human{

        }

        static class Woman extends Human{

        }
        public void sayHello (Human guy){
            System.out.println("hello, guy");
        }

        public void sayHello(Man guy){
            System.out.println("hello, gentleman");
        }

        public void sayHello(Woman guy){
            System.out.println("hello, lady");
        }

        public static void main(String[] args){
            Human man = new Man();
            Human woman = new Woman();
            StaticDispatch sr = new StaticDispatch();
            sr.sayHello(man);
            sr.sayHello(woman);
        }
    }

    运行结果是：

    hello, guy
    hello, guy

上面代码中 "Human" 称为变量的静态类型，后面的 "Man" 称为实际类型。静态类型在编译期是已知的，而实际类型在运行期才确定。编译器在重载时是通过参数的静态类型决定使用哪个重载版本。所以编译器在编译阶段，选择了 `sayHello(Human)`作为调用目标。

所有依赖静态类型来定位方法执行版本的分派动作称为静态分派。静态分派的典型应用是方法重载。静态分派发生在编译阶段。由于字面量没有显式的静态类型，他的静态类型只能通过语言上的规则去理解和推断。

## 动态分派

    public class DynamicDispatch {
        static abstract class Human{
            protected abstract void sayHello();
        }

        static class Man extends Human{

            @Override
            protected void sayHello() {
                System.out.println("man say hello");
            }
        }

        static class Woman extends Human{

            @Override
            protected void sayHello() {
                System.out.println("woman say hello");
            }
        }

        public static void main(String[] args){
            Human man = new Man();
            Human woman = new Woman();
            man.sayHello();
            woman.sayHello();
            man = new Woman();
            man.sayHello();
        }
    }

    运行结果
    man say hello
    woman say hello
    woman say hello

上面代码是典型的多态，从字节码中可以看到，每个`sayHello()`方法都是用`invokevirtual`指令调用的，后面的符号引用也是一样的，但是执行的结果却是不同的。

    INVOKEVIRTUAL com/java/DynamicDispatch$Human.sayHello ()V

原因就要从 `invokevirtual`指令的多态查找过程开始说起：

1. 找到操作数栈顶的第一个元素所指向的对象的实际类型，记作C。
2. 如果在类型C中找到与常量描述符和简单名称都相符的方法，则进行访问权限校验，如果通过则返回该方法的直接引用，查找过程结束；如果不通过，则返回`java.lang.IllegalAccessError`异常。
3. 否则，按照继承关系从下往上依次对C的各个父类进行第2步的搜索和验证过程。
4. 如果始终没有找到合适的方法，则抛出`java.lang.AbstractMethodError`异常。

由于`invokevirtual`执行的第一步就是在运行期确定接受者的实际类型，所以调用中的`invokevirtual`指令把常量池中的类方法符号引用解析到了不同的直接引用上，这个过程就是方法重写的本质。

这种在运行期根据实际类型确定方法执行版本的分派过程称为动态分派。

