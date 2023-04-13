### 线程池

#### 为什么要用线程池？

**使用线程池的好处：**

- **降低资源消耗：** 通过重复利用已创建的线程降低线程创建和销毁造成的消耗。

- **提高响应速度：** 当任务到达时，任务可以不需要的等到线程创建就能立即执行

- **提高线程的可管理性：** 线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控


#### 线程池ThreadPoolExecutor重要参数

![markdown](https://ddmcc-1255635056.file.myqcloud.com/e23a8ac8-16f7-4715-8c62-3638898e6cb2.png)


**ThreadPoolExecutor 3 个最重要的参数：**

 - **`corePoolSize`** : 核心线程数线程数定义了最小可以同时运行的线程数量。

- **`maximumPoolSize`** : 当队列中存放的任务达到队列容量的时候，当前可以同时运行的线程数量变为最大线程数。

- **`workQueue`**: 当新任务来的时候会先判断当前运行的线程数量是否达到核心线程数，如果达到的话，新任务就会被存放在队列中。

**ThreadPoolExecutor其他常见参数:**

 - `keepAliveTime:` 当线程池中的线程数量大于 `corePoolSize` 的时候，如果这时没有新的任务提交，核心线程外的线程不会立即销毁，而是会等待，直到等待的时间超过了 keepAliveTime才会被回收销毁；

 - `unit` : `keepAliveTime` 参数的时间单位

- `threadFactory` : `executor` 创建新线程的时候会用到

- `handler`: 拒绝策略


#### 线程池threadFactory用来做什么？

线程创建的工厂，新的线程都是由 `ThreadFactory` 创建的，自定义 `ThreadFactory` 线程创建工厂可以自定义线程的创建，如修改线程组、名、优先级等


#### jdk都有哪些线程池？你是怎么选择的？

`Executors` 类封装了4个创建线程池的工厂方法。内部还是通过调用 `ThreadPoolExecutor` 构造函数来创建的：

- **FixedThreadPool**：该方法返回一个固定线程数量的线程池。该线程池中的线程数量始终不变。当有一个新的任务提交时，线程池中若有空闲线程，则立即执行。若没有，则新的任务会被暂存在一个任务队列中，待有线程空闲时，便处理在任务队列中的任务

- **SingleThreadExecutor**：方法返回一个只有一个线程的线程池。若多余一个任务被提交到该线程池，任务会被保存在一个任务队列中，待线程空闲，按先入先出的顺序执行队列中的任务

- **CachedThreadPool**：该方法返回一个可根据实际情况调整线程数量的线程池。线程池的线程数量不确定，但若有空闲线程可以复用，则会优先使用可复用的线程。若所有线程均在工作，又有新的任务提交，则会创建新的线程处理任务。所有线程在当前任务执行完毕后，将返回线程池进行复用

- **ScheduledThreadPool**：创建一个定长线程池，支持定时及周期性任务执行

**关于线程池的选择：**

《阿里巴巴 Java 开发手册》中 **强制线程池不允许使用 `Executors` 去创建**，而是通过 `ThreadPoolExecutor` 的方式，这样的处理方式规避资源耗尽的风险

>Executors 返回线程池对象的弊端如下：
>
>FixedThreadPool 和 SingleThreadExecutor (使用无界队列)： 允许请求的队列长度为 Integer.MAX_VALUE ，可能堆积大量的请求，从而导致 OOM
>
>CachedThreadPool 和 ScheduledThreadPool ： 允许创建的线程数量为 Integer.MAX_VALUE ，可能会创建大量线程，从而导致 OOM


**所以一般推荐通过 `ThreadPoolExecutor` 构造的方式自己来创建**


#### 介绍一下线程池可选择的队列

- `ArrayBlockingQueue`：有界阻塞队列，基于数组的先进先出（FIFO），其构造必须指定大小。其并发控制采用 `ReentrantLock` 来控制，不管是插入操作还是读取操作，都需要获取到锁才能进行操作。当队列容量满时，尝试将元素放入队列将导致操作阻塞;尝试从一个空队列中取一个元素也会同样阻塞。**默认情况下不能保证线程访问队列的公平性**


- `LinkedBlockingQueue`：底层基于单向链表实现的阻塞队列，可以当做无界队列也可以当做有界队列来使用，同样满足 FIFO 的特性，与 `ArrayBlockingQueue` 相比起来具有更高的吞吐量，为了防止 `LinkedBlockingQueue` 容量迅速增，损耗大量内存。通常在创建 `LinkedBlockingQueue` 对象时，会指定其大小，如果未指定，容量等于 `Integer.MAX_VALUE`

- `SynchronizedQueue`：

- `PriorityBlockingQueue`：是一个支持优先级的无界阻塞队列。默认情况下元素采用自然顺序进行排序，也可以通过自定义类实现 `compareTo()` 方法来指定元素排序规则，或者初始化时通过构造器参数 `Comparator` 来指定排序规则。并发控制采用的是 `ReentrantLock`，队列为无界队列，内部是使用平衡二叉树实现的，遍历不保证有序


#### 说说线程池拒绝策略，怎么自定义拒绝策略？

**内置四种拒绝策略：**

- CallerRunsPolicy （用主线程执行）

- AbortPolicy (**`默认`**)（抛弃任务并抛出异常）

- DiscardOldestPolicy （抛弃等待队列第一个任务，将当前任务加入）

- DiscardPolicy （放弃任务并且什么都不做）


**自定义拒绝策略**

实现 `RejectedExecutionHandler` 类，实现 `rejectedExecution()` 方法，并设置线程池的拒绝策略为自定义的策略 （策略模式）


#### 线程池被创建后里面有线程吗？

**线程池被创建后如果没有任务过来，里面是不会有线程的**。如果需要预热的话可以调用下面的两个方法：

- preStartCoreThread()：创建一个线程

- preStartAllCoreThread()：创建所有线程


#### 核心线程数会被回收吗？需要什么设置？

**核心线程数默认是不会被回收的** ，如果需要回收核心线程数，可以调用 `allowCoreThreadTimeOut(true)， 该值默认为 false`


#### 如何合理设置核心线程数的大小？

核心线程数的设置主要取决于业务是IO密集型还是CPU密集型。

- CPU密集型指的是任务主要使用来进行大量的计算，没有什么导致的线程阻塞。一般这种场景的线程池核心线程数设置为CPU核心数+1。

- IO密集型：当执行任务需要大量的io，比如磁盘io，网络io，可能会存在大量的阻塞，所以在IO密集型任务中使用多线程可以大大地加速任务的处理。一般核心线程数设置为 2*CPU核心数

java中用来获取CPU核心数的方法是： `Runtime.getRuntime().availableProcessors()`


#### 说说submit和execute两个方法有什么区别？

`submit()` 和 `execute()` 都可以往线程池中提交任务，区别是使用 `execute()` 执行任务无法获取到任务执行的返回值，而使用 `submit()` 方法， 可以使用 `Future` 来获取任务执行的返回值


#### 为什么线程池要使用阻塞队列？

因为线程一旦任务执行完之后，如果想让线程不退出，只能 **阻塞或者自旋** 来保证线程不会退出，阻塞会让cpu资源，但是自旋不会，所以为了防止线程退出和减少cpu的消耗，选择使用阻塞队列来保证线程不会退出


#### 调用了shutdownNow或者shutdown，线程一定会退出么？

这个是不一定的，因为线程池会调用线程的 `interrupt()` 来打断线程的执行，但是这个方法不会打断正在运行的线程，只对正在阻塞等待的线程生效，一旦线程执行的任务类似于一个死循环，那么任务永远不会执行完，那么线程永远都不会退出