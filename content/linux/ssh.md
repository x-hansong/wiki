---
title: ssh
layout: page
date: 2015-11-26
---
[TOC]

## 基本用法
    ssh user@host
    ssh -p port user@host

## known__hosts
第一次连接到某个主机,会提示对方的公钥,例如:

    The authenticity of host 'host (12.18.429.21)' can't be established.
    RSA key fingerprint is 98:2e:d7:e0:de:9f:ac:67:28:c2:42:2d:37:16:58:4d.
    are you sure you want to continue connecting (yes/no)?

这段话的意思是，无法确认host主机的真实性，只知道它的公钥指纹，问你还想继续连接吗？

如果输入 `yes` 确认登录的话,系统会把当前 `host` 存入 `known__hosts`,下次再连接这个主机就不会再提示.

## 公钥无密码登录
首先生成公私钥,用 `ssh-keygen`,运行成功后,在 `$HOME/.ssh` 中生成 id_rsa.pub(公钥)和id_rsa(私钥)

接着,输入以下命令把公钥传到主机上.

    ssh-copy-id user@host

之后可以直接输入 `ssh user@host` 无密码直接登录.

如果失败,打开主机上的 `/etc/ssh/sshd_config`,检测以下几行的 `#` 注释是否取消.

    RSAAuthentication yes
    PubkeyAuthentication yes
    AuthorizedKeysFile .ssh/authorized_keys
然后重启 ssh 服务

    // ubuntu系统
    service ssh restart
    // debian系统
    /etc/init.d/ssh restart

## authorized_keys
远程主机将用户的公钥，保存在登录后的用户主目录的$HOME/.ssh/authorized_keys文件中。公钥就是一段字符串，只要把它追加在authorized_keys文件的末尾就行了。

这里不使用上面的ssh-copy-id命令，改用下面的命令，解释公钥的保存过程：

    ssh user@host 'mkdir -p .ssh && cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub

这条命令由多个语句组成，依次分解开来看：

- `ssh user@host`，表示登录远程主机;
- 单引号中的`mkdir .ssh && cat >> .ssh/authorized_keys`，表示登录后在远程shell上执行的命令;
- `mkdir -p .ssh` 的作用是，如果用户主目录中的.ssh目录不存在，就创建一个;
- `'cat >> .ssh/authorized_keys' < ~/.ssh/id_rsa.pub` 的作用是，将本地的公钥文件~/.ssh/id_rsa.pub，重定向追加到远程文件authorized_keys的末尾。

写入authorized_keys文件后，公钥登录的设置就完成了。

*参考文章*
[SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html)
