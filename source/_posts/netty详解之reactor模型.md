title: netty详解之reactor模型
date: 2016-09-23 17:41:41
tags:
- netty
- io
- reactor
---

假设在办理各种证件时分为填表，审核，制作3个过程，每个过程用时10分钟，这样一个工作人员需要30分钟办理一个证件。那么有没有办法提供效率，减少等待时间呢。可以让一个专门的工作人员，每个顾客到来时就负责让顾客填表，在顾客填好表后交给其他工作人员审核。这样其他功能人员的工作效率就从30分钟提高到了20分钟。

## Reactor模式
Reactor模式就是这样一种机制，利用事件驱动减少工作线程的等待时间。Reactor模式是处理并发I/O比较常见的一种模式，用于同步I/O，中心思想是将所有要处理的I/O事件注册到一个中心I/O多路复用器上，同时主线程阻塞在多路复用器上；一旦有I/O事件**准备就绪**(区别在于多路复用器是边沿触发还是水平触发)，多路复用器返回并将相应I/O事件分发到对应的处理器中。

## 单线程模型

这是最简单的单Reactor单线程模型。Reactor线程是个多面手，负责多路分离套接字，Accept新连接，并分派请求到处理器链中。该模型适用于处理器链中业务处理组件能快速完成的场景。不过这种单线程模型不能充分利用多核资源，所以实际使用的不多。 

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/reactor3.png)


## 多线程模型（单Reactor） 

相比上一种模型，该模型在事件处理器（Handler）链部分采用了多线程（线程池），也是后端程序常用的模型。 

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/reactor4.png)


## 多线程模型（多Reactor） 

这个模型比起第二种模型，它是将Reactor分成两部分，mainReactor负责监听并accept新连接，然后将建立的socket通过多路复用器（Acceptor）分派给subReactor。subReactor负责多路分离已连接的socket，读写网络数据；业务处理功能，其交给worker线程池完成。通常，subReactor个数上可与CPU个数等同。

![](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/reactor5.png)  

## 服务端通信时序

![服务端通信序列图](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/reactor1.png)

## 客户端通信时序

![客户端通信序列图](http://raw.githubusercontent.com/minotaursu/minotaursu.github.io/source/images/reactor2.png)

Netty的IO线程NioEventLoop由于聚合了多路复用器Selector，可以同时并发处理成百上千个客户端Channel，由于读写操作都是非阻塞的，这就可以充分提升IO线程的运行效率，避免由于频繁IO阻塞导致的线程挂起。这从根本上解决了传统同步阻塞IO一连接一线程模型，架构的性能、弹性伸缩能力和可靠性都得到了极大的提升。
