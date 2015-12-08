---
title: web目录的权限配置
layout: page
date: 2015-12-09
---
[TOC]

web 目录的权限设置事关服务器的安全,遵循最小权限原则.假设 web 根目录为 `www`

1. 如果要通过 ftp 上传文件,可将 `www` 目录权限给 ftp 用户.假设用户为 `ftp`

        chmod -R www ftp

2. 配置文件夹访问权限为 755

        _ find www -type d -exec chmod 755 {} \;

3. 配置文件访问权限为 644

        _ find www -type f -exec chmod 644 {} \;
