
#### BIO、NIO、AIO有什么区别？

- **BIO属于同步阻塞IO模型** ：同步阻塞IO，其实就是服务端创建一个 `ServerSocket`， 然后就是客户端用一个 `Socket` 去连接服务端的那个 `ServerSocket`， `ServerSocket` 接收到了一个的连接请求就创建一个 `Socket` 和一个线程去跟那个 `Socket` 进行通讯。接着客户端和服务端就进行阻塞式的通信，客户端发送一个请求，服务端 `Socket` 进行处理后返回响应，在响应返回前，客户端那边就阻塞等待，什么事情也做不了。 这种方式的缺点， 每次一个客户端接入，都需要在服务端创建一个线程来服务这个客户端
- 
- **AIO是异步IO模型** ：异步 `IO` 是基于事件和回调机制实现的，也就是应用操作之后会直接返回，不会堵塞在那里，当后台处理完成，操作系统会通知相应的线程进行后续的操作