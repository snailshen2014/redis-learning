### 什么是IO多路复用
IO多路复用是一种同步IO模型，实现一个线程可以监视多个文件句柄；一旦某个文件句柄就绪，就能够通知应用程序进行相应的读写操作；没有文件句柄就绪时会阻塞应用程序，交出cpu。多路是指网络连接，
复用指的是同一个线程

### 为什么需要多路复用
没有IO多路复用机制时，有BIO、NIO两种实现方式，但有一些问题。
* 同步阻塞（BIO）
 
 服务端采用单线程，当accept一个请求后，在recv或send调用阻塞时，将无法accept其他请求（必须等上一个请求处recv或send完），无法处理并发
```
// 伪代码描述
while(1) {
  // accept阻塞
  client_fd = accept(listen_fd)
  fds.append(client_fd)
  for (fd in fds) {
    // recv阻塞（会影响上面的accept）
    if (recv(fd)) {
      // logic
    }
  }  
}

```

服务器端采用多线程，当accept一个请求后，开启线程进行recv，可以完成并发处理，但随着请求数增加需要增加系统线程，大量的线程占用很大的内存空间，并且线程切换会带来很大的开销，
10000个线程真正发生读写事件的线程数不会超过20%，每次accept都开一个线程也是一种资源浪费.


```
// 伪代码描述
while(1) {
  // accept阻塞
  client_fd = accept(listen_fd)
  // 开启线程read数据（fd增多导致线程数增多）
  new Thread func() {
    // recv阻塞（多线程不影响上面的accept）
    if (recv(fd)) {
      // logic
    }
  }  
}

```


* 同步非阻塞（NIO）
