# Interview

## 1,线程池四种拒绝策略
当任务源源不断的过来，而我们的系统又处理不过来的时候，我们要采取的策略是拒绝服务。
ThreadPoolExecutor类实现了ExecutorService接口和Executor接口，可以设置线程池corePoolSize，最大线程池大小，AliveTime，拒绝策略等
- CallerRunsPolicy：它直接在 execute 方法的调用线程中运行被拒绝的任务
这个策略显然不想放弃执行任务。但是由于池中已经没有任何资源了，那么就直接使用调用该execute的线程本身来执行。
（对某些应用场景来讲，很有可能造成当前线程也被阻塞。如果所有线程都是不能执行的，很可能导致程序没法继续跑了。需要视业务情景而定吧。）
- AbortPolicy：处理程序遭到拒绝将抛出运行时 RejectedExecutionException 
这种策略直接抛出异常，丢弃任务。（jdk默认策略，队列满并线程满时直接拒绝添加新任务，并抛出异常，为了保证后续任务的正常进行，丢弃一些也是可以接收的，记得做好记录）
- DiscardPolicy：不能执行的任务将被删除 
这种策略和AbortPolicy几乎一样，也是丢弃任务，只不过他不抛出异常。
- DiscardOldestPolicy：如果执行程序尚未关闭，则位于工作队列头部（最旧）的任务将被删除，然后重试执行程序（如果再次失败，则重复此过程） 
在pool没有关闭的前提下首先丢掉缓存在队列中的最早的任务，然后重新尝试运行该任务。这个策略需要适当小心。

## 2,jvm调优

### 1,java内存模型，新生代老年代算法

### 2,新生代老年代属于堆还是栈，还知道哪些gc算法

### 3,创建大对象也是在新生代分配吗

### 4,新生代的算法

### 5,说说minor gc,minor gc触发条件

### 6,老年代用了什么算法

### 7,说说标记整理算法

### 8,full gc是否真正回收了废弃对象

### 9,有哪些gc策略。你觉得你的项目中如果需要jvm调优你会注重哪个分带的调优，或者说更注重哪种gc调优，为什么，具体怎么做

## 3,tomcat调优

## 4,sql左连接右连接区别
左连接返回左边所有记录和右边符合条件的记录
右连接返回右边所有记录和左边符合条件的记录

## 5,stringbuffer和stringbuild区别
StringBuffer 中大部分都是synchronized，同步方法，因此是线程安全的。
StringBuild 则没有，所以它不是线程安全的。
在单线程情况下，StringBuild效率更高，因为不是同步方法。

## 6,StringBuild线程不安全体现在哪
因为StringBuild的操作都是非同步的，而它的字符数组以及长度都是继承父类的全局变量
而且操作不具有原子性，所以就有可能发生线程不安全。

## 7,你对线程安不安全怎么理解（关键词:全局变量、JVM运行时数据区、可见性、原子性、锁
>线程安全问题都是由全局变量及静态变量引起的。

这是因为JVM运行时数据区包括了程序计数器，本地方法栈，jvm栈，堆。在这四个区中，前三个都是线程间隔离的。
只有堆内存是线程间共享的。而全局变量放在堆内存中，各线程内jvm栈只保存了对象引用，所以各线程更改的还是一个
内存地址的数据。
在JDK1.8中元数据区取代了永久代，元数据区并不在虚拟机中，而在本地内存中。静态变量是保存在元数据区中的。所以对于
线程来说，操作的还是同一内存地址上的数据。

>若每个线程中对全局变量、静态变量只有读操作，而无写操作，一般来说，这个全局变量是线程安全的；
>若有多个线程同时执行写操作，一般都需要考虑线程同步，否则的话就可能影响线程安全。

- 可见性问题：jvm有一个主内存，每个线程有自己的高速缓存，当运行时每个线程都会在自己的高速缓存中建立一个变量副本，
操作完后再把值写入到主内存。在多线程的情况下，有可能当线程A操作完，但值还未写入，这时线程B获取时间片在执行变量值还是未改变的。

**线程A改变了值，线程B没有立即看到线程A修改的值这就是可见性问题。**

- 原子性问题： **即一个操作或者多个操作 要么全部执行并且执行的过程不会被任何因素打断，要么就都不执行。**
比如：size += 1 相当于 size = size + 1，这边就有两个操作了，先size+1，再把值赋给size。如果这两个操作不具备原子性，就会造成线程不安全。
在多线程情况下，有可能线程A刚做完size+1的操作，这时被线程B抢走了时间片，而B读到的就是错误的数据。

- 有序性问题： **即程序执行的顺序按照代码的先后顺序执行**  但是有时候并不是如此。

指令重排序： **源代码中的顺序与得到的字节码顺序不一样或者字节码顺序与实际的执行顺序不一样**
在编译的时候静态编译器会.java编译成.class，动态编译器jit会将.class编译成jvm执行的文件，在编译的时候还会对性能进行优化，
这时候就有可能改变了代码的顺序，即源代码和执行代码的顺序是不一致的。

#### 如何保证线程安全
- 上面说了线程安全是因为全局变量及静态变量引起的。所以把全局变量和静态变量改成局部变量就不会出现线程安全的问题。
因为局部变量的值保存在jvm栈内存里。
- 用同步锁（synchronized，Lock）可以解决原子性问题，因为线程间是互斥的，所以能保证原子性。
- 用同步锁还可以保证可见性问题，在synchronized关键字中，获得锁后会先清除工作内存的变量副本，然后从主内存拷贝变量的最新副本到线程工作内存，
执行代码，将更改后的最新的共享变量的值刷新到主内存，释放互斥锁。而Lock，在Java并发编程实战中,有句话是"线程A拿到lock对象对数据进行修改, 线程B拿到lock对象后,
对于线程A的修改线程B是可见的, 线程B可以看到线程A的所有操作结果"，也可以保证可见性。
volatile变量也可以保证该变量对所有线程可见，但不能保证原子性。
- 对于有序性synchronized和锁都可以保证，都保证了只有一个线程执行，即使重排序也不会影响结果。
- Atomic***即可以保证原子性，又可以保证可见性，底层是通过CAS和volatile实现的

#### 总结 
线程安全是因为全局变量及静态变量引起的，如果有全局变量或静态变量则要解决可见性，有序性，有序性问题。

## 8,arraylist和linkedlist区别
数据结构：ArrayList是顺序存储的数据结构，底层是数组，它的存储地址是连续的。
	  LinkedList是链表结构的线程表，地址不是连续的，它是双向链表，节点保存着对上下节点的引用。
效率：ArrayList随机查询的时间复杂度是O(1),LinkedList随机查询需要移动指针查询,复杂度为O(n)
直接添加复杂度都为O(1),但是ArrayList有可能触发扩容，需要将数组复制到新的数组。
指定位置添加ArrayList需要将添加索引后面的移动且有可能触发扩容,而LinkedList需要先查找添加位置的节点,
所以也是O(n),但是会先判断从首节点开始查询还是尾结点,当插入位置为中间时,所需时间最长。
删除操作ArrayList删除后需要移动后面的元素所以为O(n),LinkedList需要先遍历节点所以也为O(n).

## 9,spring ioc，aop作用,原理

## 10,hashmap原理（*）
HashMap基于哈希散列表实现 ，初始化大小为16，扩容因子为0.75，HashMap总是使用2的幂作为哈希表的大小。
在取模计算时，如果模数是2的幂，那么我们可以直接使用位运算来得到结果，效率要大大高于做除法。
将键值对传递给put方法时，它调用键对象的hashCode()方法来计算hashCode，
然后找到相应的bucket位置（即数组）来储存值对象。当获取对象时，通过键对象的equals()方法找到正确的键值对，然后返回值对象。
HashMap使用链表来解决hash冲突问题，当发生冲突了，对象将会储存在链表的头节点中。HashMap在每个链表节点中储存键值对对象，
当两个不同的键对象的hashCode相同时，它们会储存在同一个bucket位置的链表中，如果链表大小超过阈值（TREEIFY_THRESHOLD,8），
链表就会被改造为树形结构。当红黑树个数小于6，将会转回为链表。

## 11,Hashtable
默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。Hashtable是线程安全的，
它的每个方法中都加入了Synchronize方法。在多线程并发的环境下，可以直接使用Hashtable，不需要自己为它的方法实现同步。

## 12,concurrentHashMap
concurrentHashMap在JDK1.5出现的，为了解决HashMap线程不安全问题和Hashtable使用synchronized导致并发性能低问题。
- 1.7中使用分段锁来提升map的并发性能。在 ConcurrentHashMap 有一个 Segment 数组，将HashMap分成多个段(默认分成16个Segment，通过hash来定位Segment的位置),将锁的颗粒度降低至一个段。
- 1.8抛弃了分段锁,利用数组+链表+红黑树来实现,对数组的每个元素来加锁,将锁的颗粒度降至每个节点(即每个桶)。
内部使用synchronized+CAS来实现并发控制。

## 13,HashMap遍历
Map的四种遍历方式
- map.entrySet() 返回整个map 的Entry<key,value>集合set。
- map.entrySet().iterator 返回整个map 的Entry<key,value>集合set.在使用迭代器进行遍历。
- map.keySet() 获得key集合进行取值。
- map.values() 获得value集合，但不能取得key。

## 14,SpringMVC和Struts2区别
- SpringMVC是基于Servlet的，Struts2是基于Filter的。
- SpringMVC是方法级别，Struts2是类级别，所以Struts2可以有全局变量。它是多线程多实例的。SpringMVC默认是单例模式
（单例，原型，request，session，global session）。
- Struts2采用值栈存储请求和相应的数据,通过OGNL读取数据,SpringMVC通过方法形参赋值。

## 15,用过什么数据库，有没有做过数据库优化?
- mysql，Oracle
- 有对数据库进行创建索引和分库分表

## 16,什么情况下要用到索引，好处是什么?
创建索引的好处是提高查询的效率,所以在需要经常查询的表中适合创建索引。

## 17,哪些字段适合建立索引，mysql索引底层数据结构了解吗?（*）
- 频繁作为查询条件的字段应该创建索引
- 查询中与其他表有关联的字段，例如外键关系
- 频繁更新的字段不适合创建索引，因为每次更新不单单是更新记录，还会更新索引，保存索引文件
- 查询中排序的字段创建索引将大大提高排序的速度
- 经常增删改的表不适合创建索引
- 数据重复且分布平均的字段，因此为经常查询的和经常排序的字段建立索引。注意某些数据包含大量重复数据，因此他建立索引就没有太大的效果，例如性别字段，只有男女，不适合建立索引

## 18,索引什么情况下会失效，联合索引abc只用了a字段，索引是否会生效

- 左模糊索引会失效，右模糊不会
- where语句中使用不等于 <> 和 !=
- 使用not in
- 使用 or，但是没有把or中所有字段加上索引
- 条件中对字段进行表达式操作(+,-等等)
- 使用is not null 判断,is null 则会使用。
- 对于多列索引，不是使用的第一部分，则不会使用索引
- 数据库字段为字符串，条件不加引号，不会使用索引

2，联合索引abc只用了a字段，索引会生效

## 19,什么情况下要用到多线程，为什么要用，好处
在实际使用中，每个请求创建新线程的服务器在创建和销毁线程上花费的时间和消耗的系统资源，
甚至可能要比花在实际处理实际的用户请求的时间和资源要多的多。除了创建和销毁线程的开销之外，
活动的线程也需要消耗系统资源。如果在一个JVM中创建太多的线程，可能会导致系统由于过度消耗内存或者“切换过度”而导致系统资源不足。
为了防止资源不足，服务器应用程序需要一些办法来限制任何给定时刻处理的请求数目，
尽可能减少创建和销毁线程的次数，特别是一些资源耗费比较大的线程的创建和销毁，尽量利用已有对象来进行服务，
这就是“池化资源”技术产生的原因。

## 20,启动多个线程，如何知道他们都运行完毕了
可以使用CountDownLatch来做计数器，设定线程初始个数，每个线程执行完后都减1，当计数为0则
都执行完了。

## 21,String为什么是不可变的
String 内部是一个char数组，它是final修饰的，意味着它是不可改变的。

## 22,重写重载
#### 方法的重载：
- 方法的重载是在同一个中方法名相同
- 形参的个数或形参数据类型或形参的顺序不同
- 方法的重载与返回值类型无关
#### 方法的重写：
- 方法的重写的前提是存在继承或者实现关系的
- 方法的方法名，返回值类型，形参必须一致
- 重写的方法访问权限不能低于被重写方法的访问权限
- 重写的方法不能抛出新的异常，或者是抛出的异常比被重写方法的大

## 23,object类有哪些方法
- clone方法，浅复制。复制出来的对象和原对象互不影响。
- getClass方法，final方法。获取对象类型
- toString方法
- finalize方法，用于释放资源。在gc回收之前调用。
- equals方法，比较内存地址
- hashCode方法
- wait方法
- notify方法
- notifyAll方法

## 24,sleep和wait区别
- sleep是Thread 类的静态本地方法，wait则是Object类的本地方法
- 当线程处于上锁时，sleep()方法不会释放对象锁，即睡眠时也持有对象锁。只会让出CPU执行时间片，并不会释放同步资源锁，
wait()方法释放对象锁。
- sleep()方法可以在任何地方使用，wait()方法则只能在同步方法或同步块中使用
- sleep()必须捕获异常，wait()方法、notify()方法和notiftAll()方法不需要捕获异常
- sleep()使线程进入阻塞状态（线程睡眠），wait()方法使线程进入等待队列（线程挂起）

## 25,实现线程的方式
- 继承Thread
- 实现Runnable接口
- 实现Callable接口

#### 区别
- 实现Callable接口的任务线程**能返回执行结果**；而实现Runnable接口的任务线程不能返回结果；
- Callable接口的call()方法**允许抛出异常**；而Runnable接口的run()方法的异常只能在内部消化，不能继续上抛；
- Callable接口支持返回执行结果，此时需要调用FutureTask.get()方法实现，此方法会阻塞主线程直到获取‘将来’结果；当不调用此方法时，主线程不会阻塞！

## 26,run，start区别
start()方法会创建一个线程,并让线程进入就绪队列等待分配cpu，分到cpu后才调用实现的run()方法。

若不使用start()直接调用run()方法，因为没有开辟新的线程，所以是同步调用。

## 27,线程有几种状态
>线程: 是cpu运行和调度的基本单位，每个线程都有各自的堆栈区,一个进程中的所有线程共享着进程的内存和资源

状态: 线程可分为5种基本的状态 **新建状态,就绪状态,运行状态,阻塞状态,死亡状态** 当执行new Thread();时，
线程还不存在,只是获得一个Thread对象，只当调用start()；方法才会创建一个线程，并加入到等待执行队列，
线程在执行过程中(遇到sleep(),wait(),挂起,IO阻塞,互斥,同步),都会进入阻塞状态,sleep()当线程阻塞时间到了自己会加入到等待执行队列,
而wait();需要锁对象里的其他线程调用notify()方法唤醒，当线程执行完所有代码或是遇到异常线程死亡。

## 28,synchronize实现原理


## 29,synchronize与lock区别

- 首先synchronized是java内置关键字，在jvm层面，Lock是个java类；
- synchronized无法判断是否获取**锁的状态**，Lock可以判断是否获取到锁；
- synchronized会自动释放锁(a 线程执行完同步代码会释放锁 ；b 线程执行过程中发生异常会释放锁)，Lock需在**finally中手工释放锁**（unlock()方法释放锁），否则容易造成线程死锁；
- 用synchronized关键字的两个线程1和线程2，如果当前线程1获得锁，线程2线程等待。如果线程1阻塞，线程2则会一直等待下去，**而Lock锁就不一定会等待下去，如果尝试获取不到锁，线程可以不用一直等待就结束了**；
- synchronized的锁**可重入、不可中断、非公平**，而Lock锁**可重入、可判断、可公平**（两者皆可）
- Lock锁适合**大量同步的代码的同步问题**，synchronized锁**适合代码少量的同步问题**。

## 30,还知道哪些锁，说说自旋锁
- 可重入锁(ReentrantLock)，再一个锁中可以获得另一个锁。
- 读写锁(ReadWriteLock)，其读锁是共享锁，其写锁是独享锁。
- 自旋锁(CAS)

#### CAS算法
- 需要读写的内存值 V
- 进行比较的值 A
- 拟写入的新值 B

当且仅当 V 的值等于 A时，CAS通过原子方式用新值B来更新V的值，否则不会执行任何操作（比较和替换是一个原子操作）

#### 自旋锁
>是指当一个线程在获取锁的时候，如果锁已经被其它线程获取，那么该线程将循环等待，然后不断的判断锁是否能够被成功获取，直到获取到锁才会退出循环。

## 31,死锁产生原因
只有满足 **互斥,不可剥夺,请求与保持,循环等待** 才有可能发生死锁现象。

[死锁产生的条件及解决](https://ddmcc.space/2019/06/26/deadlock-conditions/)

## 32,为什么java可以一次编译，到处运行
因为JVM，Java源代码编译成字节码文件，然后在经过JVM编译成机器码文件，在再执行。所以这边的到处运行时指运行在
JVM上。到处运行前提是有JVM。

## 33,事务特性
- 原子性：是指事务包含的所有操作要么全部成功，要么全部失败回滚
- 一致性：事务执行前和执行后必须处于一致性状态
- 隔离性：当多个用户并发访问数据库时，数据库为每一个用户开启的事务，不被其他事务的操作所干扰，多个并发事务之间要相互隔离
- 持久性：一个事务一旦被提交了，那么对数据库中的数据的改变就是永久性的，即便在数据库系统遇到故障的情况下也不会丢失事物的操作

## 34,mysql有哪些函数
- concat：合并字符串
- now：获取当前时间
- DATE_FORMAT：格式化日期
- max：获取最大值
- min：最小值
- sum：和
- count：获取行数
- avg：平均值

## 35,http和https区别
- https协议需要申请证书，一般免费证书较少，因而需要一定费用。

- http是超文本传输协议，信息是明文传输，https则是具有安全性的ssl加密传输协议。

- http和https使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。

- http的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比http协议安全。

## 36,get和post区别
- get请求有长度的限制,而post请求没有长度的限制
- get请求的数据是以key/value的形式跟上url的后面,而post请求是以附件的形式发送过去的
- get请求的数据会在地址栏中明文的显示,而且还会保存在浏览器的历史记录,而post请求不会，所以post的安全性更高

## 37,Spring Bean是单例吗,其中的单例bean是怎么实现的
单例在spring中是默认的，如果要产生多例，则在配置文件的bean中添加scope="prototype";

在Spring内存有一个缓存注册hash表，上面存着已经实例化好的bean，spring依赖注入时，首先从缓存中获取bean实例，
如果为null，对缓存map加锁，然后再从缓存中获取bean，如果继续为null，就创建一个bean。这样双重判断，
能够避免在加锁的瞬间，有其他依赖注入引发bean实例的创建，从而造成重复创建的结果。创建后把bean存到缓存表中。

## 38,redis有支持哪些数据结构

## 39,servlet怎么取前端参数
request.getParamter

## 40,设计个洗牌算法

## 41,jdk中random实现原理，取随机种子

## 42,抽象类，接口区别，接口中可以定义成员变量吗
- 方法：接口的方法默认是 public abstract，所有方法在接口中不能有实现(Java 8 开始接口方法可以有默认实现），抽象类可以有非抽象的方法
- 变量： 接口中的实例变量默认是public static final 类型的，而抽象类中则不一定 
- 子类实现： 一个类可以实现多个接口，但最多只能实现一个抽象类 （Java单继承多实现）
- 子类实现方法： 一个类实现接口的话要实现接口的所有方法，而抽象类不一定 
- 接口和抽象类都不能用 new 实例化，但可以声明，但是必须引用一个实现该接口的对象 
- 从设计层面来说，抽象是对类的抽象，是一种模板设计，接口是行为的抽象，是一种行为的规范

接口中可以定义成员变量，接口中的实例变量默认是public static final类型

## 43,接口中方法权限可以是private吗，接口是否可以写具体实现
不可以,JDK1.8中允许有默认实现。

## 44,sql交并集
- 用INNER JOIN查询交集
- 用UNION形成并集

## 45,事务隔离级别
- 读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。可能造成脏读。
- 读提交，顾名思义，就是一个事务要等另一个事务提交后才能读取数据。解决了脏读，但可能出现了一个事务范围内两个相同的查询却返回了不同数据，这就是不可重复读
- 重复读，就是在开始读取数据（事务开启）时，不再允许修改操作。重复读可以解决不可重复读问题，但是可能还会有幻读问题。不可重复读对应的是修改，即UPDATE操作，幻读问题对应的是插入INSERT操作。
- Serializable 是最高的事务隔离级别，在该级别下，事务串行化顺序执行，可以避免脏读、不可重复读与幻读。但是这种事务隔离级别效率低下，比较耗数据库性能，一般不使用。

**大多数数据库默认的事务隔离级别是Read committed，比如Sql Server , Oracle。Mysql的默认隔离级别是Repeatable read**

## 46,能调用另一个类的私有方法吗，怎么做?
可以
- 通过反射，获得私有方法，设置可访问，执行invoke
- 在类的内部创建一个public方法调用私有方法，然后去调用public方法

## 47,如果线程池执行shutdown或shutdownNow，线程池中线程会中断吗，会出现什么异常，catch处理中该写些什么？（*）
shutdown只是将线程池的状态设置为SHUTWDOWN状态，正在执行的任务会继续执行下去，没有被执行的则中断。
而shutdownNow则是将线程池的状态设置为STOP，正在执行的任务则被停止，没被执行任务的则返回。

## 48,用过哪些concurrent包下的类，说说原子类（*）
- ReentrantLock 可重入互斥锁，实现自Lock，需要手动获得锁，与释放锁。
- CountDownLatch 可以让一组线程同时

## 49,CAS操作可能会存在什么问题？
#### ABA
问题描述：线程t1将它的值从A变为B，再从B变为A。同时有线程t2要将值从A变为C。但CAS检查的时候会发现没有改变，但是实质上它已经发生了改变 。可能会造成数据的缺失。

解决方法：CAS还是类似于乐观锁，同数据乐观锁的方式给它加一个版本号或者时间戳，如AtomicStampedReference

#### 自旋消耗资源
问题描述：多个线程争夺同一个资源时，如果自旋一直不成功，将会一直占用CPU。

解决方法：破坏掉for死循环，当超过一定时间或者一定次数时，return退出。JDK8新增的LongAddr,和ConcurrentHashMap类似的方法。当多个线程竞争时，将粒度变小，将一个变量拆分为多个变量，达到多个线程访问多个资源的效果，最后再调用sum把它合起来

#### 多变量共享一致性问题
解决方法： CAS操作是针对一个变量的，如果对多个变量操作，1. 可以加锁来解决。2 .封装成对象类解决。

## 50,jdk中有哪些设计模式的运用，项目中用了哪些设计模式（*）
- Singleton：Runtime，NumberFormat
- Factory（静态工厂）：Class.forName，Integer.valueOf
- Factory Method（工厂方法）：Collection.iterator
- Observer（观察者）：java.util.Observer,Observable，Swing中的Listener
- Strategy（策略）：ThreadPoolExecutor中的四种拒绝策略

## 51,TreeMap中compartor用了什么设计模式
Strategy（策略模式）

## 52,有哪些单例模式的实现方式
- 懒汉式，先不实例化，需要用的时候才判断为空初始化。
在反射面前没什么用，因为java反射机制是能够实例化构造方法为private的类的；
线程不安全，并发环境可能出现多个singleton实例

- 饿汉式，类初始化的时候就立刻实例化。
系统启动时，占用资源；如果后期这个单例没有被使用，会造成资源浪费

- 线程同步式，在获取对象方法上加锁
在方法调用上加了同步，虽然线程安全了，但是每次都有同步，会影响性能

- 双重锁检查式，先判断是否为空，为空再加锁判断是否为空
优点：线程安全，且确保了只有第一次调用单例时才会做同步，避免了每次都同步的性能损耗；
缺点：双重锁降低了程序响应速度和性能

- 枚举式单例

    public enum Singleton {
        INSTANCE;

        public void getInstance() {
        }
    }

这种方式不仅能解决多线程同步问题，而且能防止反序列化重新创建新的对象，不过jdk1.5中才加入enum特性

- 注册登记式
相当于有一个容器装载所有实例，在实例产生之前先检查下容器有没有，如果有就直接取出来，如果没有就先new一个放进去，然后给后面的人用，Spring Bean容器就是一种注册登记式单例,

## 53,注解实现原理


## 54,servlet和jsp区别
jsp就是在html里面写java代码，servlet就是在java里面写html代码，其实jsp经过容器解释之后就是servlet。
jsp页面会由jsp引擎编译成.java文件，再由java编译器编译成字节码文件，再由jvm执行。所以jsp就是Servlet，
只是我们自己写代码的时候尽量能让它们各司其职，jsp更注重前端显示，servlet更注重模型和业务逻辑。

## 55,SpringMVC Controller中定义全局HashMap，它是否是线程安全的，为什么?
SpringMVC Controller默认是单例类，也可以设置成非单例。

如果是非单例的每次有请求都会是新的实例，所以它是线程安全的。

在单例多线程的情况下，如果只对变量进行读，而没有写的操作，那么可以认为线程是安全的。
如果有写的操作，那么如果不能保证线程操作的可见性，原子性，有序性。那么线程就是非安全。

## 56,IO，NIO区别，NIO原理


## 57,项目中用哪种方式解析xml的


## 58,servlet生命周期
容器通过类加载器使用servlet类对应的文件加载servlet，通过调用servlet构造函数创建一个servlet对象并调用init初始化，
当Servlet没有设置loadOnStartup时,默认是第一次调用的时候被装载并被实例化,load-on-statup的值 > 0 容器启动时实例化 
，load-on-statup = 0 启动时最后实例化。所以，每个Servlet类必须有一个公共的无参数的构造器,
初始化完成后就可以处理请求了,调用service()方法执行，doGet或doPost，当服务器关闭或重启调用destroy()销毁方法,之后Servlet实例等待回收

## 59,tomcat在初始化中做了什么事

## 60,过滤器和拦截器的区别
![](https://ws3.sinaimg.cn/large/005BYqpggy1g4jhmtlpzij30pw0duk1w.jpg)

## 61,oracle分页的sql关键字是什么
rownum  mysql：limit

## 62,分库分表按什么规则分,如果查询字段不是分库分表的规则字段，怎么办
#### 垂直分
- 垂直分库，基本的思路就是按照业务模块来划分出不同的数据库，而不是像早期一样将所有的数据表都放到同一个数据库中。
- 垂直分表，拆分是基于关系型数据库中的"列"进行的。通常情况，某个表中的字段比较多，可以新建立一张表，将不经常使用或者长度较大的字段拆分出去。

#### 水平分
- 水平分表，就是将表中不同的数据行按照一定规律分布到不同的数据库表中（这些表保存在同一个数据库中），这样来降低单表数据量，优化查询性能。
最常见的方式就是通过主键或者时间等字段进行Hash和取模后拆分。
- 水平分库分表，水平分库分表与上面讲到的水平分表的思想相同，唯一不同的就是将这些拆分出来的表保存在不同的数据库中。这也是很多大型互联网公司所选择的做法

#### 分区
- RANGE分区，字段值的范围进行分区
- HASH分区
- KEY分区

多维度查询

## 63,jdbc连接步骤
- 通过Class.forName();加载驱动类
- 用驱动管理器DriverManager.getConnection();获得连接对象
- 通过连接对象来获得执行对象Statement或preparedStatement
- 编写sql语句select,update,delete,insert
- 执行sql语句executeUpdat();或executeQuery();
- 处理返回结果,insert,update,delete返回int数代表有几行改变了
- 关闭资源,后开先关

## 64,statement和prepareStatement区别
- PreparedStatement接口代表预编译的语句，它主要的优势在于可以减少SQL的编译错误并增加SQL的安全性（减少SQL注射攻击的可能性）
- PreparedStatement中的SQL语句是可以带参数的，避免了用字符串连接拼接SQL语句的麻烦和不安全；
- 当批量处理SQL或频繁执行相同的查询时，PreparedStatement有明显的性能上的优势，由于数据库可以将编译优化后的SQL语句缓存起来，
下次执行相同结构的语句时就会很快（不用再次编译和生成执行计划）

## 65,项目中数据库事务控制你们是怎么做的

## 66,TreeMap,TreeSet,HashSet实现原理

## 67,dom4j怎么取节点
root.element(name)

## 68,为什么重写equals方法最好也得重写hashcode
- 如果两个对象相同（即用equals比较返回true），那么它们的hashCode值一定相同；

- 如果两个对象的hashCode相同，它们并不一定相同(即用equals比较返回false)

## 69,TreeMap中元素怎么排序，如果没实现comparator接口会怎么样
TreeMap默认按键的自然顺序升序进行排序，也可以传入Comparator接口进行比较器排序。

## 70,序列化作用
序列化就是一种用来处理对象流的机制，所谓对象流也就是将对象的内容进行流化（将对象转换成二进制）。
可以对流化后的对象进行读写操作，也可将流化后的对象传输于网络之间，序列化是为了解决在对对象流进行读写操作时所引发的问题。
把对象转换为字节序列的过程称为对象的序列化，把字节序列恢复为对象的过程称为对象的反序列化。

## 71,你对hash的理解

## 72,HashSet是怎么去重的
首先根据对象hashcode得到元素的存储位置，如果位置上已经有了元素则用equals比较两个对象是都相同，相同则不添加。

## 73,socket长连接短连接，连接出现异常你是怎么处理的

## 74,mybatis分页
- 数组分页，查询所有符合的值，在根据分页来subList
- Sql语句进行分页，使用limit
- RowBounds 分页

## 75,BIO,NIO,AIO区别

## 76,1.假如servlet处理一个请求需要0.4秒，那么处理100请求大概需要多久，为什么。同时处理1000个请求导致服务器压力过大崩溃怎么解决

## 77,线上系统造成服务器cpu占用率过高问题

## 78,Arraylist当前容量是10，且有9个数据，那么是添加第10个数据时扩容还是第11个
添加第11个的时候扩容

## 79,Arraylist扩容怎么实现的，为什么采用复制数组的方式而不是往后直接添加数据
Arraylist扩容是复制数组。

数组的地址是连续的，可能原来的数组后面的内存地址已经被使用了。

## 80,锁升级:偏向锁->轻量级锁->重量级锁

## 81, 线程池有哪些参数，各代表什么意思
- corePoolSize：线程池启动后，在池中保持的线程的最小数量。需要说明的是线程数量是逐步到达corePoolSize值的。例如corePoolSize被设置为10，而任务数量只有5，则线程池中最多会启动5个线程，而不是一次性地启动10个线程
- maxinumPoolSize：线程池中能容纳的最大线程数量，如果超出，则使用RejectedExecutionHandler拒绝策略处理
- keepAliveTime：线程的最大生命周期。这里的生命周期有两个约束条件：一：该参数针对的是超过corePoolSize数量的线程；二：处于非运行状态的线程。举个例子：如果corePoolSize（最小线程数）为10，maxinumPoolSize（最大线程数）为20，而此时线程池中有15个线程在运行，过了一段时间后，其中有3个线程处于等待状态的时间超过keepAliveTime指定的时间，则结束这3个线程，此时线程池中则还有12个线程正在运行。
- unit：这是keepAliveTime的时间单位，可以是纳秒，毫秒，秒，分钟等
- workQueue：任务队列。当线程池中的线程都处于运行状态，而此时任务数量继续增加，则需要一个容器来容纳这些任务，这就是任务队列。这个任务队列是一个阻塞式的单端队列

## 82,jdk中提供了哪几种线程池的实现
- newFixedThreadPool
创建一个指定工作线程数量的线程池。每当提交一个任务就创建一个工作线程，如果工作线程数量达到线程池初始的最大数，则将提交的任务存入到池队列中
- newCachedThreadPool
创建一个可缓存的线程池。这种类型的线程池特点是： 
1).工作线程的创建数量几乎没有限制(其实也有限制的,数目为Interger. MAX_VALUE), 这样可灵活的往线程池中添加线程。 
2).如果长时间没有往线程池中提交任务，即如果工作线程空闲了指定的时间(默认为1分钟)，则该工作线程将自动终止。终止后，如果你又提交了新的任务，则线程池重新创建一个工作线程
- newSingleThreadExecutor
创建一个单线程化的Executor，即只创建唯一的工作者线程来执行任务，如果这个线程异常结束，会有另一个取代它，
保证顺序执行。单工作线程最大的特点是可保证顺序地执行各个任务，并且在任意给定的时间不会有多个线程是活动的
- newScheduleThreadPool
创建一个定长的线程池，而且支持定时的以及周期性的任务执行，类似于Timer

## 83,线程池中提供哪些队列
- ArrayBlockingQueue：规定大小的BlockingQueue，其构造必须指定大小。其所含的对象是FIFO顺序排序的。

- LinkedBlockingQueue：大小不固定的BlockingQueue，若其构造时指定大小，生成的BlockingQueue有大小限制，不指定大小，其大小有Integer.MAX_VALUE来决定。其所含的对象是FIFO顺序排序的。

- PriorityBlockingQueue：类似于LinkedBlockingQueue，但是其所含对象的排序不是FIFO，而是依据对象的自然顺序或者构造函数的Comparator决定。

- SynchronizedQueue：特殊的BlockingQueue，对其的操作必须是放和取交替完成。

排队有三种通用策略：

直接提交。工作队列的默认选项是 SynchronousQueue，它将任务直接提交给线程而不保持它们。在此，如果不存在可用于立即运行任务的线程，则试图把任务加入队列将失败，因此会构造一个新的线程。此策略可以避免在处理可能具有内部依赖性的请求集时出现锁。直接提交通常要求无界 maximumPoolSizes 以避免拒绝新提交的任务。当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

无界队列。使用无界队列（例如，不具有预定义容量的 LinkedBlockingQueue）将导致在所有 corePoolSize 线程都忙时新任务在队列中等待。这样，创建的线程就不会超过 corePoolSize。（因此，maximumPoolSize的值也就无效了。）当每个任务完全独立于其他任务，即任务执行互不影响时，适合于使用无界队列；例如，在 Web页服务器中。这种排队可用于处理瞬态突发请求，当命令以超过队列所能处理的平均数连续到达时，此策略允许无界线程具有增长的可能性。

有界队列。当使用有限的 maximumPoolSizes时，有界队列（如 ArrayBlockingQueue）有助于防止资源耗尽，但是可能较难调整和控制。队列大小和最大池大小可能需要相互折衷：使用大型队列和小型池可以最大限度地降低 CPU 使用率、操作系统资源和上下文切换开销，但是可能导致人工降低吞吐量。如果任务频繁阻塞（例如，如果它们是 I/O边界），则系统可能为超过您许可的更多线程安排时间。使用小型队列通常要求较大的池大小，CPU使用率较高，但是可能遇到不可接受的调度开销，这样也会降低吞吐量

## 84,io中用了什么设计模式
装饰者模式

## 85,jdk1.8 lambda说说它的闭包体现在哪

## 86,Spring bean的生命周期

## 87,spring的BeanFactory和FactoryBean有什么区别

## 88,如果有一条SQL查询要十多秒，如何优化？

## 89,索引的优缺点?
#### 优点
- 通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。 
- 可以大大加快数据的检索速度，这也是创建索引的最主要的原因。 
- 可以加速表和表之间的连接，特别是在实现数据的参考完整性方面特别有意义。 
- 在使用分组和排序子句进行数据检索时，同样可以显著减少查询中分组和排序的时间。 
- 通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

#### 缺点
- 创建索引和维护索引要耗费时间，这种时间随着数据量的增加而增加。 
- 索引需要占物理空间，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，
那么需要的空间就会更大。 
- 当对表中的数据进行增加、删除和修改的时候，索引也要动态的维护，这样就降低了数据的维护速度

## 90,多态的体现？