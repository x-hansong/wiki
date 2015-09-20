---
title: "hexo折腾日记"
layout: page
date: 2015-09-20
---
[TOC]

## 折腾1:hack部署源码,自动添加CNAME文件
### 问题描述
我使用github pages的功能作为博客的免费空间,hexo来自动部署博客.hexo的deploy操作实质是清空.deploy目录,然后把public目录的文件复制进去,然后push到github上面.这样做简单粗暴,缺点是你根本没办法自己往生成的目录里面添加东西,因为每次它部署都会清空原来的东西,总不能每次部署完我再添加CNAME手动push吧.
### 解决方法
手动hack代码.首先找到部署相关的插件即`hexo-deployer-git`,部署的源代码在`lib/deployer.js`中.找到
```
    log.info('Copying files from public folder...');
    return fs.copyDir(publicDir, deployDir);
```
在`fs.copyDir`插入`fs.writeFile(deployDir+'/CNAME', 'blog.xiaohansong.com');`
