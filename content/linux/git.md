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

3. Windows 下大小写敏感设置。在windows下面将已经push到远端的文件，改变其文件名的大小写时，git默认会认为文件没有发生任何改动，从而拒绝提交和推送，原因是其默认配置为大小写不敏感，故须在bash下修改配置：

    git config core.ignorecase false


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

### 撤销提交

        git reset --hard HEAD~1 //退回到上一次提交
        git push --force  //将本地更改强制上传到服务器