### Spring Boot

#### 什么是SpringBoot自动装配？

`SpringBoot` 定义了一套接口规范，这套规范规定：`SpringBoot` 在启动时会扫描外部引用 `jar` 包中的META-INF/spring.factories文件，将文件中配置的类型信息加载到 `Spring` 容器（此处涉及到 JVM 类加载机制与 Spring 的容器知识），并执行类中定义的各种操作。对于外部 `jar` 来说，只需要按照 `SpringBoot` 定义的标准，就能将自己的功能装置进 `SpringBoot`


#### SpringBoot的是如何实现自动装配的？如何按需加载？


1. `Spring Boot` 通过 `@EnableAutoConfiguration` 开启自动装配。默认 `spring.boot.enableautoconfiguration=true` ，可在 `application.properties` 或 `application.yml` 中设置

2. 获取EnableAutoConfiguration注解中的 `exclude` 和 `excludeName`，用于排除指定的类

3. 通过 `SpringFactoriesLoader` 读取所有 `starter` 下 `META-INF/spring.factories` 里配置的需要自动装配的配置类并加载

4. 这些配置类会通过 `@ConditionalOnXXX` 按需加载注解的判断，只有满足的才会生效
