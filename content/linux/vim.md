---
title: vim
layout: page
date: 2015-11-25
---
[TOC]

## 替换命令

    :s/well/good/ 替换当前行第一个 well 为 good
    :n,$s/well/good/ 替换第 n 行开始到最后一行中每一行的第一个 well 为 good. n 为数字，若 n 为 .，表示从当前行开始到最后一行
    :%s/well/good/（等同于 :g/well/s//good/） 替换每一行的第一个 well 为 good
    :%s/well/good/g（等同于 :g/well/s//good/g） 替换每一行中所有 well 为 good

    可以使用 # 作为分隔符，此时中间出现的 / 不会作为分隔符
    :s#well/#good/# 替换当前行第一个 well/ 为 good/
    :%s#/usr/bin#/bin#g


## 常用快捷键

- **%**：跳转匹配的括号
