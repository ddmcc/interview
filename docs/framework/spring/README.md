### Spring

#### 谈谈你对IoC的理解？

**IoC（Inversion of Control:控制反转）** 指的是对象创建（实例化、管理）的控制权交给外部环境（Spring 框架、IoC 容器）。将对象之间的相互依赖关系交给 `IoC` 容器来管理，并由 `IoC` 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 `IoC` 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的

在 `Spring` 中， `IoC` 容器是 `Spring` 用来实现 `IoC` 的载体， `IoC` 容器实际上就是个 Map（key，value），Map 中存放的是各种对象。`Spring` 时代我们一般通过 XML 文件来配置 Bean，后来开发人员觉得 XML 文件来配置不太好，于是 `SpringBoot` 注解配置就慢慢开始流行起来
              
      
#### SpringAOP和AspectJ有什么区别?                                                          

**Spring AOP 属于运行时增强，而 AspectJ 是编译时增强。**  `Spring AOP` 基于代理(Proxying)，而 `AspectJ` 基于字节码操作(Bytecode Manipulation)。`Spring AOP` 已经集成了 `AspectJ`。`AspectJ` 相比于 `Spring AOP` 功能更加强大，但是 `Spring AOP` 相对来说更简单，如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择 `AspectJ` ，它比 `Spring AOP` 快很多

#### 说一下bean的生命周期？

1. 找到配置文件中 `Spring Bean` 的定义，其实就是 beanDefi

2. 利用反射创建一个 `Bean` 的实例，如果涉及到一些属性值利用 `set()` 方法设置者些属性值

3. 如果 `Bean` 实现了 `*.Aware` 接口，则调用相应的方法，并传入相关参数

4. 如果有和加载这个 `Bean` 的 `Spring` 容器相关的前置处理器，则调用处理器

5. 如果 `Bean` 实现了 `InitializingBean` 接口，执行 `afterPropertiesSet()` 方法

6. 如果 `Bean` 在配置文件中的定义包含 `init-method` 属性，执行指定的方法

7. 如果有和加载这个 `Bean` 的 Spring 容器相关的后置处理器，则调用处理器

8. 当要销毁 `Bean` 的时候，如果 `Bean` 实现了 `DisposableBean` 接口，执行 `destroy()` 方法，如果 `Bean` 在配置文件中的定义包含 `destroy-method` 属性，执行指定的方法


![](https://images.xiaozhuanlan.com/photo/2019/b5d264565657a5395c2781081a7483e1.jpg)


#### bean的作用域有哪些?

- `singleton` : 唯一 bean 实例，Spring 中的 bean 默认都是单例的，对单例设计模式的应用。
- `prototype` : 每次获取都会创建一个新的 bean 实例。
- `request` : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP request 内有效。
- `session` : 每一次 HTTP 请求都会产生一个新的 bean，该 bean 仅在当前 HTTP session 内有效。
- `global-session` ： 全局 session 作用域，仅仅在基于 portlet 的 web 应用中才有意义，Spring5 已经没有了。


#### 单例bean的线程安全问题了解吗？

单例 `bean` 存在线程问题，主要是因为当多个线程操作同一个对象的时候是存在资源竞争的

常见的有两种解决办法：

- 在 `bean` 中尽量避免定义可变的成员变量
- 在类中定义一个 `ThreadLocal` 成员变量，将需要的可变成员变量保存在 ThreadLocal 中（推荐的一种方式）


#### @Component和@Bean的区别是什么？


- `@Component` 注解作用于类，而 `@Bean` 注解作用于方法
- `@Component` 通常是通过类路径扫描来自动侦测以及自动装配到 `Spring` 容器中（我们可以使用 `@ComponentScan` 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 `Spring` 的 `bean` 容器中）。`@Bean` 注解通常是我们在标有该注解的方法中定义产生这个 `bean`,告诉了 `Spring` 这是某个类的实例，帮我注册到容器中
- `@Bean` 注解比 `@Component` 注解的自定义性更强，而且很多地方我们只能通过 `@Bean` 注解来注册bean。比如当我们引用第三方库中的类需要装配到 `Spring` 容器时，则只能通过 `@Bean` 来实现


#### Spring事务有几种使用方式？

- **编程式事务管理**

通过 `TransactionTemplate` 或者 `TransactionManager` 手动管理事务，`TransactionTemplate` 是Spring提供对 `TransactionManager`的封装模板类，所以通常直接使用 `TransactionTemplate`


```java

@Autowired
private TransactionTemplate transactionTemplate;

public void testTransaction() {

        transactionTemplate.execute(new TransactionCallbackWithoutResult() {
            @Override
            protected void doInTransactionWithoutResult(TransactionStatus transactionStatus) {

                try {

                    // ....  业务代码
                } catch (Exception e){
                    //回滚
                    transactionStatus.setRollbackOnly();
                }

            }
        });
}
```


- **声明式事务管理**

推荐使用这种方式，代码侵入性最小，实际是通过 `AOP` 实现（基于@Transactional 的全注解方式使用最多）

```java
@Transactional(rollback = Exception.class)
public void aMethod {
    //do something
}
```
``

#### 有哪些事务失效的场景？

1. **方法被不是public或被final/static修饰**

首先 `Spring` 有限制事务方法必须为 `public` （代码中有判断是否为public，因为JDK动态代理必须通过Java接口，只能支持public级别的方法，AOP不得不取消非public方法的支持） ，其次事务基于动态代理来实现，如果方法非 `public` 则有可能在代理类中无法重写。 `final/static` 修饰的方法同理，代理类无法重写它增加事务功能


2. **类内部方法调用**

方法拥有事务能力是因为 `Spring` 为其生成了代理类，并注册到容器中，如果直接调用则相当于 `this.xxx()` ，并不是用代理对象调用，所以没有事务功能

>解决方法：获取被Spring管理的bean来调用，比如：使用依赖注入在当前类中注入自己、使用依赖查找从容器中获取


3. **多线程调用**

因为 `spring` 的事务是通过数据库连接来实现，而数据库连接 `spring` 是放在 `threadLocal` 里面。在多线程场景下，拿到的数据库连接是不一样的，即是属于不同事务


4. **业务自己捕获了异常**

`spring` 事务只有捕捉到了业务抛出去的异常，才能进行后续的处理，如果业务自己捕获了异常，则事务无法感知

>将事务抛出或者手动事务回滚 TransactionAspectSupport.currentTransactionStatus().setRollbackOnly()

5. **抛出受检异常**

`spring` 默认只会回滚非检查异常和 `error` 异常，可以配置 `rollbackFor` 来解决


#### @Resource和@Autowired注解有什么区别


- **相同点**

`@Resource` 和 `@Autowired` 都可以作为注入属性的修饰，在接口仅有单一实现类时，两个注解的修饰效果相同，可以互相替换，不影响使用


- **不同点**

1. 注解出处不同：`@Resource` 是 `JDK` 原生的注解，`@Autowired` 是 `Spring2.5` 引入的注解

2. 参数不同：`@Resource` 有name和type等7个属性，`Autowried` 只有 `required` 属性

3. 作用范围不同：`@Resource` 能作用在类、成员变量和方法上，`Autowried` 可以作用在构造器、方法、参数、成员变量和注解上

4. 注入策略不同：

`@Autowired` 按类型装配依赖对象，默认情况下它要求依赖对象必须存在，如果允许null值，可以设置它required属性为false。如果我们想使用按名称装配，可以结合 `@Qualifier` 注解一起使用

`@Resource` 有两个中重要的属性：name和type。name属性指定byName，如果没有指定name属性，当注解标注在字段上，即默认取字段的名称作为bean名称寻找依赖对象，当注解标注在属性的setter方法上，即默认取属性名作为bean名称寻找依赖对象。需要注意的是，
`@Resource`如果没有指定name属性，并且按照默认的名称仍然找不到依赖对象时， @Resource注解会回退到按类型装配。但一旦指定了name属性，就只能按名称装配了。

**@Resource装配顺序**
　　
1. 如果同时指定了name和type，则从Spring上下文中找到唯一匹配的bean进行装配，找不到则抛出异常　　
2. 如果指定了name，则从上下文中查找名称（id）匹配的bean进行装配，找不到则抛出异常　　
3. 如果指定了type，则从上下文中找到类型匹配的唯一bean进行装配，找不到或者找到多个，都会抛出异常　　
4. 如果既没有指定name，又没有指定type，则自动按照byName方式进行装配；如果没有匹配，则回退为一个原始类型进行匹配，如果匹配则自动装配；


#### Spring框架中用到了哪些设计模式？

- **工厂模式** ：Spring 使用工厂模式通过 `BeanFactory`、`ApplicationContext` 创建bean对象
- **单例模式** ：Spring 中的 Bean 默认都是单例的
- **代理模式** ：Spring AOP 功能的实现
- **模板方法模式** : Spring 中 `jdbcTemplate`、`hibernateTemplate` 等以 `Template` 结尾的对数据库操作的类，它们就使用到了模板模式
- **观察者模式** : Spring 事件驱动模型就是观察者模式很经典的一个应用。
- **适配器模式** : Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller
