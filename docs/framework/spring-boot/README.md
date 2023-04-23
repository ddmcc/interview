### Spring Boot

#### 什么是SpringBoot自动装配？

`SpringBoot` 定义了一套接口规范，这套规范规定：`SpringBoot` 在启动时会扫描外部引用 `jar` 包中的META-INF/spring.factories文件，将文件中配置的类型信息加载到 `Spring` 容器（此处涉及到 JVM 类加载机制与 Spring 的容器知识），并执行类中定义的各种操作。对于外部 `jar` 来说，只需要按照 `SpringBoot` 定义的标准，就能将自己的功能装置进 `SpringBoot`


#### SpringBoot的是如何实现自动装配的？如何按需加载？


1. `Spring Boot` 通过 `@EnableAutoConfiguration` 开启自动装配。默认 `spring.boot.enableautoconfiguration=true` ，可在 `application.properties` 或 `application.yml` 中设置

2. 获取EnableAutoConfiguration注解中的 `exclude` 和 `excludeName`，用于排除指定的类

3. 通过 `SpringFactoriesLoader` 读取所有 `starter` 下 `META-INF/spring.factories` 里配置的需要自动装配的配置类并加载

4. 这些配置类会通过 `@ConditionalOnXXX` 按需加载注解的判断，只有满足的才会生效


#### SpringBoot的启动过程？

- 1. 通过 `SpringFactoriesLoader` 加载SpringBoot包下 `META-INF/spring.factories` ⽂件，获取并创建 `SpringApplicationRunListener` 对象
- 2. 然后由 `SpringApplicationRunListener` 来发出 `starting` 启动中事件消息
- 3. 根据 `run` 方法参数创建参数对象，并用其创建应⽤将要使⽤的 `Environment`
- 4. 完成之后，依然由 `SpringApplicationRunListener` 来发出 `environmentPrepared` 消息
- 5. 打印banner
- 6. 创建并初始化 `ApplicationContext` 上下文对象，并将listeners、environment、applicationArguments、banner等重要组件与上下文对象关联
- 7. `refreshContext` 方法，加载外部starter的spring.factories，bean的实例化等核心工作
- 8. 由 `SpringApplicationRunListener` 来发出 `started` 消息，完成最终的程序的启动
- 9. 获取所有实现启动事件监听bean（CommandLineRunner、ApplicationRunner），回调run方法
- 10. 由 `SpringApplicationRunListener` 来发出 `running` 消息，告知程序已运⾏起来