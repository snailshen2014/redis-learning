### Redis I/O模型

Linux 中的 IO 多路复用机制是指一个线程处理多个 IO 流，就是我们经常听到的 select/epoll 机制。简单来说，在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听套接字和已连接套接字。内核会一直监听这些套接字上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。

下图就是基于多路复用的 Redis IO 模型。图中的多个 FD 就是刚才所说的多个套接字。Redis 网络框架调用 epoll 机制，让内核监听这些套接字。此时，Redis 线程不会阻塞在某一个特定的监听或已连接套接字上，也就是说，不会阻塞在某一个特定的客户端请求处理上。正因为此，Redis 可以同时和多个客户端连接并处理请求，从而提升并发性。

![io](https://github.com/snailshen2014/redis-learning/blob/master/%E7%BD%91%E7%BB%9C%E6%A8%A1%E5%9E%8B/redis-io.jpg)

为了在请求到达时能通知到 Redis 线程，select/epoll 提供了基于事件的回调机制，即针对不同事件的发生，调用相应的处理函数。那么，回调机制是怎么工作的呢？其实，select/epoll 一旦监测到 FD 上有请求到达时，就会根据FD找到事件循环队列里的相应的事件。这样一来，Redis 无需一直轮询是否有请求实际发生，这就可以避免造成 CPU 资源浪费。同时，Redis 在对事件队列中的事件进行处理时，会调用相应的处理函数，这就实现了基于事件的回调。因为 Redis一直在执行事件循环，事件循环中又在检测FD中的事件，所以能及时响应客户端请求，提升 Redis 的响应性能。



### 源码分析

网络事件起点：server.c->initServer()方法里在server socket(bind,listen)后，注册了acceptTcp的事件等待客户端的连接

* 1 accept事件
```
 /* Create an event handler for accepting new connections in TCP and Unix
     * domain sockets. */
    for (j = 0; j < server.ipfd_count; j++) {
        if (aeCreateFileEvent(server.el, server.ipfd[j], AE_READABLE,
            acceptTcpHandler,NULL) == AE_ERR)
            {
                serverPanic(
                    "Unrecoverable error creating server.ipfd file event.");
            }
    }
    
    在acceptTcpHandler里，会注册socket read事件
    
```
  //启动事件循环,等待注册的网络事件发生
  server.c->main()方法里
  
  aeMain(server.el);
  
  
  ###参考
  https://blog.csdn.net/chosen0ne/article/details/42717571
  
  https://blog.csdn.net/chosen0ne/article/details/43053915

https://time.geekbang.org/column/article/270474
