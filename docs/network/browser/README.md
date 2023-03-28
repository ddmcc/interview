
### 浏览器

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