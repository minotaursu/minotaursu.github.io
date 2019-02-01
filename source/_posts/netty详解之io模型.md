title: netty详解之io模型
tags:
- nio
- java
- netty
date: 2016-09-23 16:12:54
---

提起IO模型首先想到的就是同步，异步，阻塞，非阻塞这几个概念。每个概念的含义，解释，概念间的区别这些都是好理解，这里深入*nix系统讲一下IO模型。  

在*nix中将IO模型分为5类。  
1. Blocking I/O   
2. Nonblocking I/O  
3. I/O Multiplexing (select and poll)  
4. Signal Driven I/O (SIGIO)  
5. Asynchronous I/O (the POSIX aio_functions)  

## 阻塞 I/O（blocking IO）

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/bio.png)

如图所示，系统调用recvfrom，内核kernel等待数据数据准备完成，在数据准备完成后将数据从内核态拷贝到用户态，recvfrom直到整个过程结束后才完成，在整个过程中经历2次阻塞。

## 非阻塞 I/O（nonblocking IO）

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/nio.png)

如图所示，系统调用recvfrom，内核kernel在数据没有准备完成时直接返回，系统会不断轮询，在kernel准备完成数据后将数据从内核态拷贝到用户态，在等待数据完成的过程中并不阻塞。

## I/O 多路复用（ IO multiplexing）

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/mio.png)

如图所示，IO multiplexing 使用select，poll，epoll等实现单个kernel的进程/线程处理多个IO请求，IO复用将等待数据准备和将数据拷贝给应用这两个阶段分开处理，让一个线程（而且是内核级别的线程）来处理所有的等待，一旦有相应的IO事件发生就通知继续完成IO操作，虽然仍然有阻塞和等待，但是等待总是发生在一个线程，这时使用多线程可以保证其他线程一旦唤醒就是处理数据。

## 信号驱动 I/O (Signal Driven I/O)

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/sio.png)

如图所示，系统调用recvfrom试图读取数据，并且直接返回，不管是否有数据可读，内核线程读完数据，给发信号通知应用线程，应用线程收到信息，等待内核线程将数据拷贝给应用线程。


## 异步 I/O（asynchronous IO）

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/aio.png)

如图所示，系统调用aio_read，内核kernel直接返回，系统不需要阻塞，继续做其他事情。kernel则进行等待数据准备完成，并将数据拷贝到用户态后，发送signal信号通知系统已经完成。

## 各个IO模型的对比

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/dio.png)

