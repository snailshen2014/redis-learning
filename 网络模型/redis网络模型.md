###代码

```
server.c->initServer()方法里在server socket(bind,listen)后，注册了acceptTcp的事件(FileEvent)

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
