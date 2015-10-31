---
title: spice
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


## spice-vdagent使用
> add at 2015-09-23
### 功能:
- 客户端鼠标模式
- 根据客户端分辨率调整宿机的分辨率
- 支持从客户端传输到宿机
- 支持粘贴板共享
### 构成:
- 支持vdagent的spice客户端(目前只有window版本支持)
- 客户机驱动
    - [windows驱动][1]
    - [Linux驱动][2]
### 使用方法：
- 在启动qemu虚拟机时加入选项

    ```
    -device virtio-serial-pci,id=virtio-serial0,max_ports=16,bus=pci.0,addr=0x5 \
    -chardev spicevmc,name=vdagent,id=vdagent \
    -device \
    virtserialport,nr=1,bus=virtio-serial0.0,chardev=vdagent,name=com.redhat.spice.0
    ```

- 在虚拟机里面安装对应的驱动即可
    - windows客户机直接下载驱动安装即可
    - Linux客户机可以下载源码编译安装，或者直接通过`sudo apt-get install spice-vdagent`安装

示例：
```
qemu-kvm -hda ubuntu.raw  -vnc :2 -m 4096 -smp 4 -spice port=1235,password=123  -device virtio-serial-pci,id=virtio-serial0,max_ports=16,bus=pci.0,addr=0x5 -chardev spicevmc,name=vdagent,id=vdagent -device virtserialport,nr=1,bus=virtio-serial0.0,chardev=vdagent,name=com.redhat.spice.0 -usbdevice tablet
```
[1]: http://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-0.100.exe
[2]: http://www.spice-space.org/download/releases/spice-vdagent-0.16.0.tar.bz2
