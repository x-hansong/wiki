---
title: Linux源码编程技巧
layout: page
date: 2015-10-13
---
[TOC]

## 保留字
> add at 2015-10-13

### `inline`和`__inline__`
由于`inline`在ANSI C中并非保留字，直接使用`inline`会可能会产生冲突。例如，gcc支持保留字`inline`，由于`inline`原来不是保留字，在旧的代码中可能已经有一个变量名为`inline`，这样就会产生冲突。为了解决这个问题，gcc允许作为保留字使用的`inline`前后都加上`__`,所以，`__inline__`等价于保留字`inline`.同样的道理,`__asm__`等价于保留字`asm`


## 宏定义技巧
> add at 2015-10-13

### do-while单次循环

    #define DUMP_WRITE(addr,nr) do { memcpy(bufp,addr,nr); bufp += nr; } while(0)
上面是取自`fs/proc/kcore.c`的一个宏.为什么要定义成这种形式呢?能不能定义成下面这样

    #define DUMP_WRITE(addr,nr)  memcpy(bufp,addr,nr); bufp += nr; 
这样是不行的,在一个if语句中,这样会有错误产生.例如说:

    if (addr)
        DUMP_WRITE(addr,nr);
    else
        do();
经过预处理就会变成

    if (addr)
        memcpy(bufp,addr,nr); bufp += nr;
    else
        do();

显然,这样编译会出错,因为没有括号包住if后面的两个语句.

通过上面这个例子,我们可以发现,使用do-while单次循环的方式定义,就可以避免出现类似问题.

### 空操作
> add at 2015-10-13

由于linux要兼容不同的CPU,所以在某些条件下要把一些宏定义为空操作,但是不能留空白.

    #define prepare_to_switch() do {  } while(0)
这样子定义,不管在什么条件调用这个函数都不会出现错误,如果定义成这样:

    #define prepare_to_switch()
那么在上面的if语句中就会出现错误.
