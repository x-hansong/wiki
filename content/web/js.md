---
title: js 学习笔记
layout: page
date: 2016-07-23
---
[TOC]

## js 类型
JavaScript的值可以分为两种类型：基本类型和对象类型，基本类型有：String, Number, Boolean, Symbol, undefined 和 null.，对象类型有Function, Object 和 Array.

基本类型和对象类型的区别在于可变性和比较的方式以及程序中传值。

基本类型是不可变的，换种说法就是它们的值不能改变。对比而言，对象类型是可变的，它们的值可以更新和改变。

基本类型可以按值比较，当我们把一个基本类型赋值给另外一个基本类型，是复制了一个值。而对象这是通过引用进行比较，引用的是什么呢？引用的是底层对象。当我们赋值一个对象给另一个对象时。引用指针就创建了。在这个情况下，改变一个对象的值将更新另外一个对象的值。

当我们尝试在基本类型的值中调用方法时，JavaScript使用包装对象来临时控制基本类型，导致对象变为只读的并在垃圾回收后执行。

### Date
在JavaScript中，Date对象用来表示日期和时间。

要获取系统当前时间，用：

    var now = new Date();
    now; // Wed Jun 24 2015 19:49:22 GMT+0800 (CST)
    now.getFullYear(); // 2015, 年份
    now.getMonth(); // 5, 月份，注意月份范围是0~11，5表示六月
    now.getDate(); // 24, 表示24号
    now.getDay(); // 3, 表示星期三
    now.getHours(); // 19, 24小时制
    now.getMinutes(); // 49, 分钟
    now.getSeconds(); // 22, 秒
    now.getMilliseconds(); // 875, 毫秒数
    now.getTime(); // 1435146562875, 以number形式表示的时间戳

*注意*:
- 当前时间是浏览器从本机操作系统获取的时间，所以不一定准确，因为用户可以把当前时间设定为任何值。
- JavaScript的月份范围用整数表示是0~11，0表示一月，1表示二月……，所以要表示6月，我们传入的是5！

## 注意事项
- 不要使用`new Number()`、`new Boolean()`、`new String()`创建包装对象；
- 用`parseInt()`或`parseFloat()`来转换任意类型到`number`；
- 用`String()`来转换任意类型到`string`，或者直接调用某个对象的`toString()`方法；
- 通常不必把任意类型转换为`boolean`再判断，因为可以直接写`if (myVar) {...}`；
- `typeof`操作符可以判断出`number`、`boolean`、`string`、`function`和`undefined`；
- 判断`Array`要使用`Array.isArray(arr)`；
- 判断`null`请使用`myVar === null`；
- 判断某个全局变量是否存在用`typeof window.myVar === 'undefined'`；
- 函数内部判断某个变量是否存在用`typeof myVar === 'undefined'`。

最后有细心的同学指出，任何对象都有`toString()`方法吗？`null`和`undefined`就没有！确实如此，这两个特殊值要除外，虽然`null`还伪装成了`object`类型。

更细心的同学指出，`number`对象调用`toString()`报SyntaxError：

    123.toString(); // SyntaxError
遇到这种情况，要特殊处理一下：

    123..toString(); // '123', 注意是两个点！
    (123).toString(); // '123'
不要问为什么，这就是JavaScript代码的乐趣！
