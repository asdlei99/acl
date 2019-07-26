# 通过 http 协议上传文件
## 概述
本例子使用 acl 库的 HTTP 库及服务器框架编写，其中服务器模型采用 acl 中基于 IO 事件的线程池模型，该服务模型既可以编写出完全非阻塞服务（通常编写时需要注意一些细节），也可以编写完全阻塞服务或者半非阻塞服务。得益于 IO 事件引擎，当客户端连接空闲时，该连接不会占用任何线程，而是被置入 IO 事件集合中，只有当该连接有数据可读时，服务框架才会将一个线程与该连接绑定。  
本示例是一个半非阻塞服务模型，当框架读取 HTTP 客户端请求头时是阻塞的，而在读取 HTTP 客户端上传的 MIME 数据体时是非阻塞的，这样做是有好处的，当用户上传的文件比较大时，可能会耗费较长时间，如果使一个线程与该客户端连接长期绑定，则有可能会耗尽线程池中的线程资源。本示例中，因为读 HTTP 数据体过程是非阻塞的，意味着当连接“暂时”没有数据时，线程便会与该 HTTP 连接“解绑”而被“归还”至线程池中，成为空闲线程，当该连接又有新数据到达时线程池中便会有线程被选定来处理该连接上传的数据。这样，线程池中的宝贵的线程资源基本达到了按需使用的目的，仅需少量线程便可以处理大量的 HTTP 并发请求。   
