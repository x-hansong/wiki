---
title: Velocity 实践
layout: page
date: 2017-03-16
---
[TOC]

## Velocity 转义字符
Velocity 语法不支持转义，也就是说不能用`\"`是没用的。但是有的时候我们是需要在字符串中输入`"`的，例如说拼接 json 字符串的时候，这时候就需要用到 Velocity Tools `EscapeTool`来进行转义。

```
VelocityContext context = new VelocityContext();
context.put("esc", new EscapeTool());

-------------------------------------------------
#set($test="${esc.q}name${esc.q}")
$test
```
输出结果：
```
"name"
```

其他转义方式参考文档[EscapeTool](http://velocity.apache.org/tools/devel/apidocs/org/apache/velocity/tools/generic/EscapeTool.html)