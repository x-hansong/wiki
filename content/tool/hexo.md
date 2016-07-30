---
title: "hexo折腾日记"
layout: page
date: 2015-09-20
---
[TOC]

## 折腾1:hack部署源码,自动添加CNAME文件
> add at 2015-09-20
### 问题描述
我使用github pages的功能作为博客的免费空间,hexo来自动部署博客.hexo的deploy操作实质是清空.deploy目录,然后把public目录的文件复制进去,然后push到github上面.这样做简单粗暴,缺点是你根本没办法自己往生成的目录里面添加东西,因为每次它部署都会清空原来的东西,总不能每次部署完我再添加CNAME手动push吧.
### 解决方法
手动hack代码.首先找到部署相关的插件即`hexo-deployer-git`,部署的源代码在`lib/deployer.js`中.找到
```
    log.info('Copying files from public folder...');
    return fs.copyDir(publicDir, deployDir);
```
在`fs.copyDir`上面插入`fs.writeFile(deployDir+'/CNAME', 'blog.xiaohansong.com');`

## 文章置顶
> add at 2016-07-30

在需要置顶文章的Front-matter中加上top: true即可，但是默认文章置顶在某一页而不是首页。所以需要修改排序的源码。

将`node_modules/hexo-generator-index/lib/generator.js`修改为：

    'use strict';
    var pagination = require('hexo-pagination');
    module.exports = function(locals){
      var config = this.config;
      var posts = locals.posts;
        posts.data = posts.data.sort(function(a, b) {
            if(a.top && b.top) { // 两篇文章top都有定义
                if(a.top == b.top) return b.date - a.date; // 若top值一样则按照文章日期降序排
                else return b.top - a.top; // 否则按照top值降序排
            }
            else if(a.top && !b.top) { // 以下是只有一篇文章top有定义，那么将有top的排在前面（这里用异或操作居然不行233）
                return -1;
            }
            else if(!a.top && b.top) {
                return 1;
            }
            else return b.date - a.date; // 都没定义按照文章日期降序排
        });
      var paginationDir = config.pagination_dir || 'page';
      return pagination('', posts, {
        perPage: config.index_generator.per_page,
        layout: ['index', 'archive'],
        format: paginationDir + '/%d/',
        data: {
          __index: true
        }
      });
    };

> 参考[解决Hexo置顶问题](http://www.netcan666.com/2015/11/22/%E8%A7%A3%E5%86%B3Hexo%E7%BD%AE%E9%A1%B6%E9%97%AE%E9%A2%98/)