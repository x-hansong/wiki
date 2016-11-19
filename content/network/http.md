---
title: HTTP 协议
layout: page
date: 2016-08-18
---
[TOC]

## HTTP 请求
1. 请求行：方法字段+URL字段+HTTP协议版本字段
2. 头部行
3. 内容

## 请求方法（Request Method）

HTTP/1.1协议中共定义了八种方法（有时也叫“动作”）来表明Request-URI指定的资源的不同操作方式：

- OPTIONS：返回服务器针对特定资源所支持的HTTP请求方法。也可以利用向Web服务器发送’*’的请求来测试服务器的功能性。
- HEAD：向服务器索要与GET请求相一致的响应，只不过响应体将不会被返回。这一方法可以在不必传输整个响应内容的情况下，就可以获取包含在响应消息头中的元信息。
- GET：向特定的资源发出请求。注意：GET方法不应当被用于产生“副作用”的操作中，例如在Web 应用程序中。其中一个原因是GET可能会被网络蜘蛛等随意访问。
- POST：向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。
- PUT：向指定资源位置上传其最新内容。
- DELETE：删除指定资源。
- TRACE：回显服务器收到的请求。
- CONNECT：HTTP/1.1协议中预留给能够将连接改为管道方式的代理服务器。

## HTTP 响应
1. 状态行：HTTP版本+状态码+状态码描述
2. 头部行
3. 返回内容

### 状态码
状态码分类：

1. 信息类 (100-199)
2. 响应成功 (200-299)
3. 重定向类 (300-399)
4. 客户端错误类 (400-499)
5. 服务端错误类 (500-599)

[状态码](http://www.w3school.com.cn/tags/html_ref_httpmessages.asp)

- 200 OK 服务器成功处理了请求
- 301 Moved Permanently（重定向）请求的URL已移走。Response中应该包含一个Location URL, 说明资源现在所处的位置
- 302 Found（已找到）与状态码301类似。但这里的移除是临时的。 客户端会使用Location中给出的URL，重新发送新的HTTP request
- 304 Not Modified（未修改）客户的缓存资源是最新的， 要客户端使用缓存
- 400 Bad Request（坏请求）告诉客户端，它发送了一个错误的请求。
- 401 Unauthorized（未授权）被请求的页面需要用户名和密码。
- 403 Forbidden 对被请求页面的访问被禁止。
- 404 Not Found 未找到资源
- 500 Internal Server Error服务器遇到一个错误，使其无法对请求提供服务
- 501 Not Implemented   请求未完成。服务器不支持所请求的功能。
- 502 Bad Gateway 请求未完成。服务器从上游服务器收到一个无效的响应。
- 503 Service Unavailable 请求未完成。服务器临时过载或当机。
- 504 Gateway Timeout 网关超时。

## HTTP 浏览器缓存支持
[浏览器缓存机制](http://www.cnblogs.com/skynet/archive/2012/11/28/2792503.html)