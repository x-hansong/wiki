---
title: git
layout: page
date: 2015-01-04
---
[TOC]

## git 配置
1. 设置记住 github 的用户密码，避免重复输入密码

        git config --global credential.helper wincred

2. 设置 push 默认推送给远程同名的分支

        git config --global push.default current


## git 操作

### 分支
1. 切换远程分支

        git checkout -b gh-pages origin/gh-pages

2. 查看所有分支

        git branch -va

3. 新建分支

        git checkout -b issue

4. 推送本地分支到远程仓库

        git push origin issue


### 远程仓库

1. 切换远程仓库链接

        git remote -v         //查看远程仓库链接
        git remote set-url origin https://github.com/USERNAME/OTHERREPOSITORY.git
        git remote set-url origin git@github.com:USERNAME/OTHERREPOSITORY.git