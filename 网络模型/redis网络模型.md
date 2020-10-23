### Redis I/O模型

Linux 中的 IO 多路复用机制是指一个线程处理多个 IO 流，就是我们经常听到的 select/epoll 机制。简单来说，在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听套接字和已连接套接字。内核会一直监听这些套接字上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个 IO 流的效果。

下图就是基于多路复用的 Redis IO 模型。图中的多个 FD 就是刚才所说的多个套接字。Redis 网络框架调用 epoll 机制，让内核监听这些套接字。此时，Redis 线程不会阻塞在某一个特定的监听或已连接套接字上，也就是说，不会阻塞在某一个特定的客户端请求处理上。正因为此，Redis 可以同时和多个客户端连接并处理请求，从而提升并发性。

![io](https://github.com/snailshen2014/redis-learning/blob/master/%E7%BD%91%E7%BB%9C%E6%A8%A1%E5%9E%8B/redis-io.jpg)

为了在请求到达时能通知到 Redis 线程，select/epoll 提供了基于事件的回调机制，即针对不同事件的发生，调用相应的处理函数。那么，回调机制是怎么工作的呢？其实，select/epoll 一旦监测到 FD 上有请求到达时，就会根据FD找到事件循环队列里的相应的事件。这样一来，Redis 无需一直轮询是否有请求实际发生，这就可以避免造成 CPU 资源浪费。同时，Redis 在对事件队列中的事件进行处理时，会调用相应的处理函数，这就实现了基于事件的回调。因为 Redis一直在执行事件循环，事件循环中又在检测FD中的事件，所以能及时响应客户端请求，提升 Redis 的响应性能。



### 源码分析



* accept事件注册

文件server.c文件中的initServer()方法里在server socket(bind,listen)后，注册了acceptTcp的事件等待客户端的连接

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
    
```

* read读事件
在在acceptTcpHandler里,每次会处理1000个客户端连接请求，连接三次握手建立连接之后，会调用acceptCommonHandler函数，这个函数中会创建client结构，在创建client时，会为连接的socket FD注册
读事件回调方法readQueryFromClient。
```
void acceptTcpHandler(aeEventLoop *el, int fd, void *privdata, int mask) {
    int cport, cfd, max = MAX_ACCEPTS_PER_CALL;
    char cip[NET_IP_STR_LEN];
    UNUSED(el);
    UNUSED(mask);
    UNUSED(privdata);

    while(max--) {
        cfd = anetTcpAccept(server.neterr, fd, cip, sizeof(cip), &cport);
        if (cfd == ANET_ERR) {
            if (errno != EWOULDBLOCK)
                serverLog(LL_WARNING,
                    "Accepting client connection: %s", server.neterr);
            return;
        }
        serverLog(LL_VERBOSE,"Accepted %s:%d", cip, cport);
        printf("###Syj accept fd:[%d]\n",cfd);
        acceptCommonHandler(cfd,0,cip);
    }
}

static void acceptCommonHandler(int fd, int flags, char *ip) {
    client *c;
    //创建client,
    if ((c = createClient(fd)) == NULL) {
        serverLog(LL_WARNING,
            "Error registering fd event for the new client: %s (fd=%d)",
            strerror(errno),fd);
        close(fd); /* May be already closed, just ignore errors */
        return;
    }
    //...剩余代码省略
}
    //根据连接的套接字创建client结构，注册socket read事件
client *createClient(int fd) {
    client *c = zmalloc(sizeof(client));

    /* passing -1 as fd it is possible to create a non connected client.
     * This is useful since all the commands needs to be executed
     * in the context of a client. When commands are executed in other
     * contexts (for instance a Lua script) we need a non connected client. */
    if (fd != -1) {
        anetNonBlock(NULL,fd);
        // 关闭Nagle算法，提升响应速度
        anetEnableTcpNoDelay(NULL,fd);
        if (server.tcpkeepalive)
            anetKeepAlive(NULL,fd,server.tcpkeepalive);
        //注册socket读事件
        if (aeCreateFileEvent(server.el,fd,AE_READABLE,
            readQueryFromClient, c) == AE_ERR)
        {
            close(fd);
            zfree(c);
            return NULL;
        }
    }
    //剩余代码省略
}

```

* write事件

写事件是在事件循环之前beforeSleep()方法里，处理输出缓冲区时注册的，具体的代码是在handleClientsWithPendingWrites（）方法
里，主要逻辑是循环待发送缓冲区，调用底层write发送数据，发送完成后，判断是否还有待发送的数据，如果又则为fd注册发送事件。
```
//部分代码片段
if (clientHasPendingReplies(c)) {
            int ae_flags = AE_WRITABLE;
            /* For the fsync=always policy, we want that a given FD is never
             * served for reading and writing in the same event loop iteration,
             * so that in the middle of receiving the query, and serving it
             * to the client, we'll call beforeSleep() that will do the
             * actual fsync of AOF to disk. AE_BARRIER ensures that. */
            if (server.aof_state == AOF_ON &&
                server.aof_fsync == AOF_FSYNC_ALWAYS)
            {
                ae_flags |= AE_BARRIER;
            }
            //注册client reply 事件
            if (aeCreateFileEvent(server.el, c->fd, ae_flags,
                sendReplyToClient, c) == AE_ERR)
            {
                    freeClientAsync(c);
            }
        }

```

  
  
  ###参考
  https://blog.csdn.net/chosen0ne/article/details/42717571
  
  https://blog.csdn.net/chosen0ne/article/details/43053915

https://time.geekbang.org/column/article/270474
