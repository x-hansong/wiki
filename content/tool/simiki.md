---
title: "Simiki-个人Wiki写作"
layout: page
date: 2015-09-20
---
[TOC]

# Simiki #

Simiki is a simple wiki framework, written in [Python](https://www.python.org/).

* Easy to use. Creating a wiki only needs a few steps
* Use [Markdown](http://daringfireball.net/projects/markdown/). Just open your editor and write
* Store source files by category
* Static HTML output
* A CLI tool to manage the wiki

Simiki is short for `Simple Wiki` :)

## Quick Start ##

### Install ###

	pip install simiki

### Update ###

	pip install -U simiki

### Init Site ###

	mkdir mywiki && cd mywiki
	simiki init

### Create a new wiki ###

	simiki new -t "Hello Simiki" -c first-catetory

### Generate ###

	simiki generate

### Preview ###

	simiki preview

For more information, `simiki -h` or have a look at [Simiki.org](http://simiki.org)

## Others ##

* [simiki.org](http://simiki.org)
* <https://github.com/tankywoo/simiki>
* Email: <me@tankywoo.com>

## 补充

### 自动生成目录
在文中插入`[TOC]`,这个功能作者没有在文档里面提出来.可能觉得大家都知道的吧,反正我一开始是不知道的.
