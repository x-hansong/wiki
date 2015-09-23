---
title: spice学习日记
layout: page
date: 2015-09-22
---
[TOC]

## spice vdagent打开qemu虚拟I/O串口virio serial port的过程
> add at 2015-09-22

1. linux内核virio控制台驱动发送`VIRTIO_CONSOLE_PORT_OPEN`消息给qemu
2. qemu的spicevmc字符设备驱动调用`qemu_spice_add_interface`来注册spice-server的agent字符设备
3. spice-server接着调用spicevmc驱动的状态改变回调函数,使其准备开始接收数据
4. 状态回调函数发送`CHR_EVENT_OPENED`消息给virio控制台字符设备后端
5. virio控制台字符设备后端发送`VIRTIO_CONSOLE_PORT_OPEN`给linux内核的virio控制台驱动
只有当上面的5步完成之后,virio控制台驱动才认为虚拟串口处于打开状态,能够进行读写
