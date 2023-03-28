### ThreadLocal类


#### 说说ThreadLocal类？作用？数据结构？

- **作用**

`ThreadLocal` 对象可以存储线程局部变量，每个线程 `Thread` 拥有一份自己的副本变量，多个线程互不干扰

- **数据结构**

`Thread` 类有一个类型为 `ThreadLocal.ThreadLocalMap` 的实例变量 `threadLocals` ，也就是说每个线程有一个自己的 `ThreadLocalMap。`

`ThreadLocalMap` 有自己的独立实现，可以简单地将它的 `key` 视作 `ThreadLocal`，`value` 为代码中放入的值（实际上key并不是ThreadLocal本身，而是它的一个弱引用）。

每个线程在往 `ThreadLocal` 里放值的时候，都会往自己的 `ThreadLocalMap` 里存，读也是以 `ThreadLocal` 作为 `key`，在自己的 `map` 里找对应的 `key`，从而实现了线程隔离。

`ThreadLocalMap` 有点类似 `HashMap` 的结构，只是 `HashMap` 是由数组+链表实现的，而 `ThreadLocalMap` 中并没有链表结构。
它内部是由一个 `Entry` 数组来实现的， 它的key是ThreadLocal<?> k ，继承自WeakReference， 也就是我们常说的弱引用类型，值则为我们存储的值


#### ThreadLocalMap哈希算法？怎么解决hash冲突？

**hash算法**

在 `ThreadLocal` 中有一个增量属性为 `HASH_INCREMENT = 0x61c88647`， 每当创建一个ThreadLocal对象，这个ThreadLocal.nextHashCode 这个值就会增长 `0x61c88647` 。
这个值很特殊，它是 **斐波那契数** 也叫 **黄金分割数**。`hash` 增量为 这个数字，带来的好处就是 `hash` 分布非常均匀。在数组中的索引通过 `hashcode & (length - 1)` 计算得到

**怎么解决hash冲突**

虽然 `ThreadLocalMap` 中使用了 **黄金分割数** 来作为 `hash` 计算因子，大大减少了 `Hash` 冲突的概率，但是仍然会存在冲突。
`HashMap` 中解决冲突的方法是在数组上构造一个链表结构，冲突的数据挂载到链表上，如果链表长度超过一定数量则会转化成红黑树。而 `ThreadLocalMap` 中并没有链表结构，所以这里不能适用 `HashMap` 解决冲突的方式

当往 `map` 中 `set` 时，会遍历当前的数组，直到 `Entry` 为空，有几种情况：

- 通过 `hash` 计算后的槽位对应的 `Entry` 数据为空：停止遍历，直接将数据放到该槽位

- 槽位数据不为空，`key` 值与当前计算获取的key值一致：**直接更新该槽位的数据**

- 槽位数据不为空，往后遍历过程中，在找到Entry为null的槽位之前，遇到 `key = null` 的Entry: 则开始替换过期数据逻辑，从 `key = null` 的槽位开始向后遍历，进行探测式数据清理工作


#### ThreadLocalMap扩容机制？

当容量超过 `threshold = len * 2 / 3` 即超过 `2/3` 时就开始执行 `rehash()`, 先进行探测式数据清理，然后在判断容量超过 `threshold - threshold / 4` 即当前容量超过一半时执行
`resize()`，**扩容后的大小为原来的两倍**


#### ThreadLocal内存泄露问题？源码是怎么解决的？为什么要用弱引用？

- `解决key内存泄露的问题`

ThreadLocal中，获取到线程私有变量是通过线程持有的一个threadLocalMap，然后传入ThreadLocal当做key获取到对象的，这时候就有个问题，如果你在使用完 ThreadLocal 之后，将其置为null，这时候这个对象并不能被回收，
因为他还有 ThreadLocalMap -> entry -> key的引用，直到该线程被销毁，但是这个线程很可能会被放到线程池中不会被销毁，这就产生了内存泄露。jdk是通过弱引用来解决的这个问题的，
entry中对key的引用是弱引用，当你取消了ThreadLocal的强引用之后，他就只剩下一个弱引用了，所以也会被回收。**所以弱引用主要用来解决key内存泄露的问题**

>弱引用在下一次GC时会被回收，不管内存是否足够


- `解决值内存泄露`


因为key是弱引用，在外部没有强引用的情况下，下一次GC时就会被回收，就会存在(null, value)的情况，又因为还有 ThreadLocalMap -> entry -> value的引用，直到该线程被销毁，但是这个线程很可能会被放到线程池中不会被销毁，这就产生了内存泄露
源码是通过在，`get`，`set`，`remove` 等方法时去检测删除过期的 `Entry` 即 null key 的


#### ThreadLocal使用场景？

**解决线程安全问题**


**传递线程上下文**
