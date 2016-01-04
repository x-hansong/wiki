---
title: powershell
layout: page
date: 2015-01-04
---
[TOC]

## 前言
Powershell 是 Windows 下新一代的 shell。我们可以使用 [scoop](http://scoop.sh/) 来管理软件包，它可以让 PS 用起来像 linux shell 一样，可以使用大部分 GNU 工具。另外，还有一个 PS 模块管理工具 [PsGet](http://psget.net/)，可以安装 PS 脚本工具。

## scoop 的安装与使用
1. 安装

    打开 powershell，输入：

        iex (new-object net.webclient).downloadstring('https://get.scoop.sh')

2. 允许脚本执行

        set-executionpolicy unrestricted -s cu

3. 安装软件包

        scoop install curl

### 常用软件

#### PS 样式美化
1. 安装 config

        scoop install concfg

2. 导入 Solarized 主题

        concfg import solarized

3. 安装 pshazz，提供类似 linux shell 的 prompt，并且支持 git 分支显示和命令补全。

        scoop install pshazz

#### GNU 工具
scoop 提供许多 GNU 工具，例如说 curl，vim 等。可以直接 `scoop install vim` 安装。

#### 其他软件
scoop 的软件仓库是 [bucket](https://github.com/lukesampson/scoop/tree/master/bucket)，可以查看然后选择想要的软件进行安装。


## PsGet 的安装与使用
1. 安装

    打开 powershell，输入：

        (new-object Net.WebClient).DownloadString("http://psget.net/GetPsGet.ps1") | iex

2. 安装模块

        Install-Module PsUrl

### 常用模块
PsGet 的模块仓库是 [modules](http://psget.net/directory/)

#### go 书签模块
    Install-Module go

[使用指南](https://github.com/cameronharp/Go-Shell)