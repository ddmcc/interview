- Java基础
  - [String类和常量池](?id=String类和常量池)
    - [String对象的两种创建方式](?id=String对象的两种创建方式)
    - [newString这句话创建了几个对象？](?id=newString这句话创建了几个对象？)
  - [为什么说反射速度慢？为什么慢？](?id=为什么说反射速度慢？为什么慢？)
  - [==与equals的区别](?id=等号与equals的区别)
  - [hashCode()与equals()的相关规定](?id=hashCode与equals的相关规定)
  
- 设计模式
  - [说说设计模式6大原则](?id=说说设计模式6大原则)
  - [代理模式和桥接模式的异同点？](?id=代理模式和桥接模式的异同点？)
  - [有几种单例的实现方式？以及它们的优缺点？](?id=有几种单例的实现方式？以及它们的优缺点？)

- 集合容器
  - [集合框架底层数据结构总结](?id=集合框架底层数据结构总结)
  - [使用什么集合你是怎么选择的？](?id=使用什么集合你是怎么选择的？)
  - [为什么要使用集合？而不是数组](?id=为什么要使用集合？而不是数组)
  
  - [List](?id=List)
    - [ArrayList和Vector的区别](?id=ArrayList和Vector的区别)
    - [ArrayList与LinkedList区别](?id=ArrayList与LinkedList区别)
    - [RandomAccess接口的作用是什么？](?id=RandomAccess接口的作用是什么？)
    - [说一说ArrayList的扩容机制吧](?id=说一说ArrayList的扩容机制吧)
    - [说说ensureCapacity方法的作用](?id=说说ensureCapacity方法的作用)
    - [System.arraycopy()和Arrays.copyOf()方法](?id=arraycopy和copyof方法)

  - [Set](?id=Set)
    - [比较HashSet、LinkedHashSet和TreeSet三者的异同](?id=比较HashSet、LinkedHashSet和TreeSet三者的异同)
    - [HashSet是怎么去重的？](?id=HashSet是怎么去重的？)

  - [Map](?id=Map)
    - [HashMap和Hashtable的区别](?id=HashMap和Hashtable的区别)
    - [HashMap底层数据结构](?id=HashMap底层数据结构)
    - [HashMap的长度为什么是2的幂次方？](?id=HashMap的长度为什么是2的幂次方？)
    - [ConcurrentHashMap底层具体实现](?id=ConcurrentHashMap底层具体实现)
    - [ConcurrentHashMap和Hashtable的区别](?id=ConcurrentHashMap和Hashtable的区别)

- [JVM](?id=JVM)

  - [类加载](?id=类加载)
    - [类的生命周期](?id=类的生命周期)
    - [类加载过程](?id=类加载过程)
    - [双亲委派加载机制](?id=双亲委派加载机制)
    - [如何实现一个自定义类加载器？](?id=如何实现一个自定义类加载器？)
    - [如果不想用双亲委派模型怎么办？](?id=如果不想用双亲委派模型怎么办？)
    - [双亲委派模型的好处？](?id=双亲委派模型的好处？)
    
  - [JVM内存区域](?id=JVM内存区域)
    - [运行时数据区](?id=运行时数据区)
    - [程序计数器为什么是私有的？](?id=程序计数器为什么是私有的？)
    - [虚拟机栈和本地方法栈为什么是私有的？](?id=虚拟机栈和本地方法栈为什么是私有的？)
    - [深拷贝与浅拷贝](?id=深拷贝与浅拷贝)
    - [堆和栈的区别？](?id=堆和栈的区别？)
    - [方法是如何调用的？](?id=方法是如何调用的？)
    - [方法区和永久代的关系](?id=方法区和永久代的关系)
    - [为什么要将永久代替换为元空间呢？](?id=为什么要将永久代替换为元空间呢？)
    
  - [内存分配与垃圾回收](?id=内存分配与垃圾回收)
    - [对象内存分配机制](?id=对象内存分配机制)
    - [动态计算晋升年龄阈值](?id=动态计算晋升年龄阈值)
    - [垃圾回收算法](?id=垃圾回收算法)
    - [为什么堆内存要分为新生代、老年代？](?id=为什么堆内存要分为新生代、老年代？)
    - [为什么要有Survivor区？](?id=为什么要有Survivor区？)
    - [新生代gc工作流程](?id=新生代gc工作流程)
    - [如何判断对象是否死亡（两种方法）](?id=如何判断对象是否死亡（两种方法）)
    - [MinorGc和FullGC 有什么不同呢？](?id=MinorGc和FullGC有什么不同呢？)
    - [什么是空间分配担保机制？](?id=什么是空间分配担保机制？)
    
  - [Java对象](?id=Java对象)
    - [对象的创建过程](?id=对象的创建过程)
    - [对象内存的分配方式？](?id=对象内存的分配方式？)
    - [对象的内存布局](?id=对象的内存布局)
    - [对象的访问定位](?id=对象的访问定位)
    - [Java中有哪些引用类型？](?id=Java中有哪些引用类型？)
    
  - [垃圾回收器](?id=垃圾回收器)
    
- IO
  
- 网络
  - [网络协议](?id=网络协议)
  
      - [TCP](?id=TCP)
        - [TCP是什么？](?id=TCP是什么？)
        - [TCP三次握手协议](?id=TCP三次握手协议)
        - [为什么要三次握手？](?id=?为什么要三次握手？)
        - [第2次握手传回了ACK，为什么还要传回SYN？](?id=第2次握手传回了ACK，为什么还要传回SYN？)
        - [四次挥手协议](?id=四次挥手协议)
        - [为什么要四次挥手？](?id=为什么要四次挥手？)
        
      - [HTTP长连接，短连接](?id=HTTP长连接，短连接)
      - [HTTP1.0和HTTP1.1的主要区别是什么?](?id=HTTP1.0和HTTP1.1的主要区别是什么?)
      - [HTTP和HTTPS的区别？](?id=HTTP和HTTPS的区别？)
      
  - [浏览器](?id=浏览器)
      - [Cookie和Session的区别](?id=Cookie和Session的区别)
      - [在浏览器中输入url地址->>显示主页的过程](?id=在浏览器中输入url地址->>显示主页的过程)
      - [Http无状态链接，如何实现session跟踪？](?id=Http无状态链接，如何实现session跟踪？)

- 并发编程

  - [什么是线程和进程？](?id=什么是线程和进程？)
  - [说说线程的生命周期和状态？](?id=说说线程的生命周期和状态？)
  - [说说sleep()方法和wait()方法区别和共同点？](?id=说说sleep方法和wai方法区别和共同点？)
  - [为什么我们不能直接调用run()方法？](?id=为什么我们不能直接调用run方法？)
  - [为什么要使用多线程呢？](?id=为什么要使用多线程呢？)
  - [使用多线程可能带来什么问题？](?id=使用多线程可能带来什么问题？)
  - [什么是线程死锁？如何避免死锁？](?id=什么是线程死锁？如何避免死锁？)
  - [说说并发与并行的区别？](?id=说说并发与并行的区别？)
  - [什么是上下文切换？](?id=什么是上下文切换？)
  - [并发编程三大特性](?id=并发编程三大特性)

  - 线程池
      - [为什么要用线程池？](?id=为什么要用线程池？)
      - [线程池ThreadPoolExecutor重要参数](?id=线程池ThreadPoolExecutor重要参数)
      - [线程池被创建后里面有线程吗？](?id=线程池被创建后里面有线程吗？)
      - [核心线程数会被回收吗？需要什么设置？](?id=核心线程数会被回收吗？需要什么设置？)

  - synchronized关键字
      - [讲一下synchronized关键字的底层原理](?id=讲一下synchronized关键字的底层原理)
      - [谈谈synchronized和ReentrantLock的区别](?id=谈谈synchronized和ReentrantLock的区别)
      - [synchronized锁升级过程](?id=synchronized锁升级过程)

  - volatile关键字
    - [volatile关键字的作用？](?id=volatile关键字的作用？)
    - [说说synchronized关键字和volatile关键字的区别？](?id=说说synchronized关键字和volatile关键字的区别？)

  - ThreadLocal类
      - [说说ThreadLocal类？作用？数据结构？](?id=说说ThreadLocal类？作用？数据结构？)
      - [ThreadLocalMap哈希算法？怎么解决hash冲突？](?id=ThreadLocalMap哈希算法？怎么解决hash冲突？)
      - [ThreadLocalMap扩容机制？](?id=ThreadLocalMap扩容机制？)
      - [ThreadLocal内存泄露问题？源码是怎么解决的？为什么要用弱引用？](?id=ThreadLocal内存泄露问题？源码是怎么解决的？为什么要用弱引用？)
      - [ThreadLocal使用场景？](?id=ThreadLocal使用场景？)
      
 
 
- 框架
  
- 数据库
  
- redis

- RPC