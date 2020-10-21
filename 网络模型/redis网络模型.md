###代码

网络事件起点：server.c->initServer()方法里在server socket(bind,listen)后，注册了acceptTcp的事件等待客户端的连接

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
