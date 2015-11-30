---
title: EPOLL在ET和LT下的读写
layout: page
date: 2015-11-30
---

epoll是Linux上一个可扩展的I/O事件通知机制。主要有两种工作模式

**Level Triggered ( LT )水平触发**，默认方式，即当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序可以不立即处理该事件。下次调用epoll_wait时，会再次响应应用程序并通知此事件。

**Edge Triggered ( ET )边缘触发**，即当epoll_wait检测到描述符事件发生并将此事件通知应用程序，应用程序必须立即处理该事件。如果不处理，下次调用epoll_wait时，不会再次响应应用程序并通知此事件。

注：ET模式在很大程度上减少了epoll事件被重复触发的次数，因此效率要比LT模式高。epoll工作在ET模式的时候，必须使用非阻塞套接口，以避免由于一个文件句柄的阻塞读/阻塞写操作把处理多个文件描述符的任务饿死。

那么对应到socket编程中的accept, read, write，有必要详细说说。

在LT模式下，accept,read,write和平时的编程方式并没有要特别注意的地方，因为只要对应的文件描述符中还有数据未读取或者处于可写状态，都会有通知。例如说，在read的时候没有一次性把缓冲区的数据全部读出来，那么epoll还会再次通知此事件。**可以在阻塞和非阻塞的套接字上使用**。

然而，在ET模式下，epoll只在事件发生时发生通知，没有新的事件发生就不通知。例如，在read的时候没有一次性把缓冲区的数据全部读出来，那么epoll不会再次通知此事件。除非有新的事件发生，不然监听的描述符是无法被触发。**只能在非阻塞的套接字上使用**。
>Edge Triggered event distribution delivers events only when events happens on the monitored file 。

ET模式下accept存在的问题
考虑这种情况：多个连接同时到达，服务器的TCP就绪队列瞬间积累多个就绪连接，由于是边缘触发模式，epoll只会通知一次，accept只处理一个连接，导致TCP就绪队列中剩下的连接都得不到处理。

解决办法是用while循环抱住accept调用，处理完TCP就绪队列中的所有连接后再退出循环。如何知道是否处理完就绪队列中的所有连接呢？accept返回-1并且errno设置为EAGAIN就表示所有连接都处理完。

	while ((conn_sock = accept(listenfd,(struct sockaddr *) &remote, (size_t *)&addrlen)) > 0) {
		 handle_client(conn_sock);
	}
	if (conn_sock == -1) {
		    if (errno != EAGAIN && errno != ECONNABORTED && errno != EPROTO && errno != EINTR)
	    perror("accept");
	}
ET模式下，读取要一次把缓冲区的数据读完

- 如果read返回0，那么说明已经接受所有数据
- 如果errno=EAGAIN，说明还有数据未接收，等待下一次通知
- 如果read返回-1，说明发生错误，停止处理

> Receiving an event from epoll_wait(2) should suggest to you that such file descriptor is ready for the requested I/O operation. You have simply to consider it ready until you will receive the next EAGAIN. When and how you will use such file descriptor is entirely up to you. Also, the condition that the read/write I/O space is exhausted can be detected by checking the amount of data read/write from/to the target file descriptor. For example, if you call read(2) by asking to read a certain amount of data and read(2) returns a lower number of bytes, you can be sure to have exhausted the read I/O space for such file descriptor. Same is valid when writing using the write(2) function.

以上是官方文档的解释，一旦得到通知说明I/O已经准备好了，不管你什么时候去处理，可以保证读取完缓冲区的数据。写也是同样道理。
