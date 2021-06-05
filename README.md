## 我的学习笔记

#### jvm运行时数据区
![markdown](https://ddmcc-1255635056.file.myqcloud.com/06512b0c-a37c-4470-8940-05b256e1d1e7.png)

#### 类加载过程


#### 双亲委派加载机制


#### 对象内存分配机制


#### 如何判断对象是否死亡（两种方法）
- 引用计数法：

给对象中添加一个引用计数器，每当有一个地方引用它，计数器就加 1；当引用失效，计数器就减 1；任何时候计数器为 0 的对象就是不可能再被使用的
    这个方法实现简单，效率高，但是目前主流的虚拟机中并没有选择这个算法来管理内存，其最主要的原因是它很难解决对象之间相互循环引用的问题
- 可达性分析算法：

基本思想就是通过一系列的称为 “GC Roots” 的对象作为起点，从这些节点开始向下搜索，节点所走过的路径称为引用链，当一个对象到 GC Roots 没有任何引用链相连的话，则证明此对象是不可用的


>可作为 GC Roots 的对象包括下面几种:
>
>虚拟机栈(栈帧中的本地变量表)中引用的对象
>
>本地方法栈(Native 方法)中引用的对象
>
>方法区中类静态属性引用的对象
>
>方法区中常量引用的对象
>
>所有被同步锁持有的对象


#### 垃圾分代回收算法


#### 被标记的对象就会被回收吗？


#### HotSpot 为什么要分为新生代和老年代？


## 网络协议

#### TCP三次握手协议

- 客户端–发送带有 SYN 标志的数据包–一次握手–服务端
- 服务端–发送带有 SYN/ACK（syn + 1） 标志的数据包–二次握手–客户端
- 客户端–发送带有带有 ACK（syn + 1） 标志的数据包–三次握手–服务端


#### 为什么要三次握手？

三次握手的目的是建立可靠的通信信道，说到通讯，简单来说就是数据的发送与接收，而三次握手最主要的目的就是双方确认自己与对方的发送与接收是正常的
 
 
#### 第2次握手传回了ACK，为什么还要传回SYN？

接收端传回发送端所发送的ACK是为了告诉客户端，我接收到的信息确实就是你所发送的信号了，这表明从客户端到服务端的通信是正常的。而回传SYN则是为了建立并确认从服务端到客户端的通信


#### 四次挥手协议

- 客户端-发送一个 FIN，用来关闭客户端到服务器的数据传送
- 服务器-收到这个 FIN，它发回一 个 ACK，确认序号为收到的序号加1 。和 SYN 一样，一个 FIN 将占用一个序号
- 服务器-关闭与客户端的连接，发送一个FIN给客户端
- 客户端-发回 ACK 报文确认，并将确认序号设置为收到序号加1


#### 为什么要四次挥手？

任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接

举个例子：A 和 B 打电话，通话即将结束后，A 说“我没啥要说的了”，B回答“我知道了”，但是 B 可能还会有要说的话，A 不能要求 B 跟着自己的节奏结束通话，于是 B 可能又巴拉巴拉说了一通，最后 B 说“我说完了”，A 回答“知道了”，这样通话才算结束


#### HTTP长连接，短连接

在HTTP/1.0中默认使用短连接。也就是说，客户端和服务器每进行一次HTTP操作，就建立一次连接，任务结束就中断连接。当客户端浏览器访问的某个HTML或其他类型的Web页中包含有其他的Web资源（如JavaScript文件、图像文件、CSS文件等），每遇到这样一个Web资源，浏览器就会重新建立一个HTTP会话

而从HTTP/1.1起，默认使用长连接，用以保持连接特性，使用长连接的HTTP协议，会在响应头加入这行代码：`Connection:keep-alive`
在使用长连接的情况下，当一个网页打开完成后，客户端和服务器之间用于传输HTTP数据的TCP连接不会关闭，客户端再次访问这个服务器时，会继续使用这一条已经建立的连接。Keep-Alive不会永久保持连接，它有一个保持时间，可以在不同的服务器软件（如Apache）中设定这个时间。实现长连接需要客户端和服务端都支持长连接

**HTTP协议的长连接和短连接，实质上是TCP协议的长连接和短连接**


#### HTTP1.0和HTTP1.1的主要区别是什么?

长连接 : 在HTTP/1.0中，默认使用的是短连接，HTTP/1.1的持续连接有非流水线方式和流水线方式流水线方式是客户在收到HTTP的响应报文之前就能接着发送新的请求报文。与之相对应的非流水线方式是客户在收到前一个响应后才能发送下一个请求

**错误状态响应码 :** 在HTTP1.1中新增了24个错误状态响应码

**带宽优化及网络连接的使用：** HTTP1.1则在请求头引入了range头域，它允许只请求资源的某个部分，即返回码是206

#### HTTP和HTTPS的区别？

**端口** ：HTTP的URL由“http://”起始且默认使用端口80，而HTTPS的URL由“https://”起始且默认使用端口443。

**安全性和资源消耗：** HTTP协议运行在TCP之上，所有传输的内容都是明文，客户端和服务器端都无法验证对方的身份。HTTPS是运行在SSL/TLS之上的HTTP协议，SSL/TLS 运行在TCP之上。所有传输的内容都经过加密，加密采用对称加密，但对称加密的密钥用服务器方的证书进行了非对称加密。所以说，HTTP 安全性没有 HTTPS高，但是 HTTPS 比HTTP耗费更多服务器资源


## 浏览器

#### Http无状态链接，如何实现session跟踪？

大部分情况下，我们都是通过在 Cookie 中附加一个 Session ID 来方式来跟踪

- Cookie 被禁用怎么办?
- 最常用的就是利用 URL 重写把 Session ID 直接附加在URL路径的后面

#### Cookie和Session的区别

- **存储位置：** Cookie 存储在客户端中，而Session存储在服务器上
- **安全性：** 相对来说 Session 安全性更高。如果要在 Cookie 中存储一些敏感信息，不要直接写入 Cookie 中，最好能将 Cookie 信息加密然后使用到的时候再去服务器端解
- **作用：** Cookie 一般用来保存用户信息比如 token，Session 的主要作用就是通过服务端记录用户的状态

#### 在浏览器中输入url地址->>显示主页的过程

1. DNS解析：获取域名对应IP
2. TCP连接：三次握手
3. 发送HTTP请求
4. 服务器处理请求并返回HTTP报文
5. 浏览器解析渲染页面
6. 连接结束