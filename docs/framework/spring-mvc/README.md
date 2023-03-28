### SpringMVC

#### SpringMVC工作原理了解吗?

![](https://img-blog.csdnimg.cn/img_convert/de6d2b213f112297298f3e223bf08f28.png)


- 客户端（浏览器）发送请求，直接请求到 `DispatcherServlet`
- `DispatcherServlet` 根据请求信息调用 `HandlerMapping`，根据请求路径匹配到对应的 `Handler`
- 解析到对应的 `Handler`（也就是我们平常说的 Controller 控制器）后，开始由 `HandlerAdapter` 适配器处理
- `HandlerAdapter` 会根据Handler来调用真正的处理器开处理请求，并处理相应的业务逻辑
- 处理器处理完业务后，会返回一个 `ModelAndView` 对象，`Model` 是返回的数据对象，View 是个逻辑上的 View。
- `ViewResolver` 会根据逻辑 View 查找实际的 View。
- `DispaterServlet` 把返回的 `Model` 传给 `View`（视图渲染）。
- 把 View 返回给请求者（浏览器）
