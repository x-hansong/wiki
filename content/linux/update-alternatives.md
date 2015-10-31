---
title: 使用update-alternatives配置多个jdk共存
layout: page
date: 2015-10-31
---
[TOC]

## update-alternatives简介
> update-alternatives是dpkg的实用工具，用来维护系统命令的符号链接，以决定系统默认使用什么命令。在Debian系统中，我们可能会同时安装有很多功能类似的程序和可选配置，如Web浏览器程序(firefox，konqueror)、窗口管理器(wmaker、metacity)和鼠标的不同主题等。这样，用户在使用系统时就可进行选择，以满足自已的需求。但对于普通用户来说，在这些程序间进行选择配置会较困难。update-alternatives工具就是为了解决这个问题，帮助用户能方便地选择自已喜欢程序和配置系统功能。

简单来说,update-alternatives可以帮助我们切换默认程序,例如说你系统里面装了多个版本的jdk,那么默认的`java`命令到底指向哪一个jdk版本?当然我们可以手动给java命令添加符号链接,然而这样有点麻烦.为了解决多版本以及多个类似程序共存,灵活切换的问题,update-alternatives就诞生了.

下面我们结合多个版本jdk共存的例子来讲解update-alternatives的用法

## 使用方法
### 修改命令的符号链接
首先,我们执行下面的命令,查看`java`命令的配置情况

    sudo update-alternatives --config java

结果如下:
    
    有 2 个候选项可用于替换 java (提供 /usr/bin/java)。

      选择       路径                                   优先级  状态
    ------------------------------------------------------------
    * 0            /usr/lib/jvm/java-8-oracle/bin/java       1088      自动模式
      1            /usr/lib/jvm/java-7-oracle/jre/bin/java   1072      手动模式
      2            /usr/lib/jvm/java-8-oracle/bin/java       1088      手动模式

    要维持当前值[*]请按回车键，或者键入选择的编号：

我们来分析一下上面的表:
可以看到,`java`有两个可选的符号链接

    /usr/lib/jvm/java-7-oracle/jre/bin/java
    /usr/lib/jvm/java-8-oracle/bin/java

它们的优先级分别是1072,1088.

- 当设置为自动模式时,系统会自动选择优先级大的链接.也就是说,现在`java`命令指向的是优先级最大的`java-8-oracle`.
- 当设置为手动模式时,则选中哪个,就使用哪个.

### 添加命令的符号链接
上面我们提到如何修改命令的符号链接,那么如果我们连一个jdk都没安装怎么办呢?

首先,我们下载jdk,然后解压到`/usr/lib/jvm/`中
然后,执行下面的命令

    sudo update-alternatives --install /usr/bin/java java /usr/lib/java-8-oracle/bin/java 1088

我们来看看这条命令做了什么,命令的格式:`--install link name path priority`

- `link`指系统命令的链接
- `name`配置名称
- `path`可选的命令链接路径
- `priority`优先级

上面这条命令给`java`的配置项添加了一个可选的链接`/usr/lib/java-8-oracle/bin/java`,并且设置优先级为1088.

当我们设置`java`为自动模式时,添加一个新的jdk就变得很简单,只要下载新的jdk,然后加入到`java`的配置项中,设置最高的优先级,系统就会自动把`java`命令指向最新的jdk中的`java`.
