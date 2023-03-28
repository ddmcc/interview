### MyBatis

#### #{}和${}的区别是什么？

- ${}是变量占位符，属于静态文本替换，在 `mybatis` 拼接sql时就会将占位符替换成相应的属性值，比如${driver}会被静态替换为com.mysql.jdbc.Driver

- #{}是 `sql` 的参数占位符，`MyBatis` 会将 sql 中的#{}替换为? 号，在 sql 执行前会使用 `PreparedStatement` 的参数设置方法，按序给 sql 的? 号占位符设置参数值，比如 ps.setInt(0, parameterValue)，#{item.name} 的取值方式为使用反射从参数对象中获取 item 对象的 name 属性值，相当于 param.getItem().getName()。



#### mapper接口的工作原理是什么？

`Mapper` 接口的全限名，就是映射文件中的 `namespace` 的值，接口的方法名，就是映射文件中的id值，及 `MappedStatement` 的 `id` 值，接口方法内的参数，就是传递给 `sql` 的参数
`Mapper` 接口是没有实现类的，当调用接口方法时，接口全限名 + 方法名拼接字符串作为 `key` 值，可定位唯一一个 `MappedStatement`，在 MyBatis 中，每一个 `<select>` 、 `<insert>` 、 `<update>` 、 `<delete>` 标签，都会被解析为一个 `MappedStatement` 对象，用于保存sql语句等

**`mapper` 接口的工作原理是 `JDK` 动态代理，`MyBatis` 运行时会使用 `JDK` 动态代理为 `mapper` 接口生成代理 `proxy` 对象，并被注入到我们要使用的地方，调用时代理对象 `proxy` 会拦截接口方法，转而执行 `MappedStatement` 所代表的 `sql`，然后将 `sql` 执行结果返回**


#### mapper接口可以重载吗？

- todo

#### MyBatis是如何进行分页的？分页插件的原理是什么？


- `MyBatis` 使用 `RowBounds` 对象进行分页，它是针对 `ResultSet` 结果集执行的内存分页，而非物理分页；
- 可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，
- 也可以使用分页插件来完成物理分页

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数


#### MyBatis的插件运行原理以及如何编写一个插件？

`MyBatis` 仅可以编写针对 `ParameterHandler` 、 `ResultSetHandler` 、 `StatementHandler` 、 `Executor` 这 4 种接口的插件，`MyBatis` 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 `InvocationHandler` 的 invoke() 方法，当然，只会拦截那些你指定需要拦截的方法。

实现 `MyBatis` 的 `Interceptor` 接口并复写 `intercept()` 方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件

>如：分页插件的实现：
@Intercepts(
    {
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
        @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
    }
)
> 指定拦截Executor类中两个query方法


#### 能简述一下动态sql的执行原理不？

其执行原理为，使用 `OGNL` 从参数对象中（执行上下文）计算表达式的值，根据表达式的值动态拼接 `sql`，以此来完成动态 `sql` 的功能。


#### MyBatis是否支持延迟加载？它的实现原理是什么？

`MyBatis` 仅支持 `association` 关联对象和 `collection` 关联集合对象的延迟加载，`association` 指的就是一对一，`collection` 指的就是一对多查询。在 `MyBatis` 配置文件中，可以配置是否启用延迟加载 `lazyLoadingEnabled` = true|false。

**它的原理是，使用 `CGLIB` 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 a.getB().getName() ，拦截器 `invoke()` 方法发现 a.getB() 是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName() 方法的调用。这就是延迟加载的基本原理**


#### MyBatis是否可以映射Enum枚举类？

MyBatis 可以映射枚举类，不单可以映射枚举类，MyBatis 可以映射任何对象到表的一列上。映射方式为自定义一个 `TypeHandler` ，实现 `TypeHandler` 的 setParameter() 和 getResult() 接口方法。 TypeHandler 有两个作用：

- 一是完成从 `javaType` 至 `jdbcType` 的转换；
- 二是完成 `jdbcType` 至 `javaType` 的转换，体现为 `setParameter()` 和 `getResult()` 两个方法，分别代表设置 `sql` 问号占位符参数和获取列查询结果

>如 mybatis-plus 的枚举映射 MybatisEnumTypeHandler 或者官方默认的EnumTypeHandler
