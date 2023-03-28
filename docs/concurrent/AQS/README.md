### AQS

[Java并发之AQS详解](https://www.cnblogs.com/waterystone/p/4920797.html)

[Java并发包基石-AQS详解](https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html)


#### 请你说一下自己对于AQS原理的理解？

**`AQS` 核心思想是，如果被请求的共享资源未被占用，则将当前请求的线程设置为占用的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制 `AQS` 是用 `CLH` 队列锁实现的，即将暂时获取不到锁的线程加入到队列中**

>CLH(Craig,Landin and Hagersten)队列（`FIFO`）是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。`AQS` 是将每条请求共享资源的线程封装成一个队列的结点（Node）来实现锁的分配

`AQS` 使用一个 `int` 成员变量来表示同步状态，通过内置的 `FIFO` 队列来完成获取资源线程的排队工作。`AQS` 使用 `CAS` 对该同步状态进行原子操作实现对其值的修改


#### AQS对资源的共享方式

**`AQS` 定义两种资源共享方式**

- `Exclusive（独占）`：只有一个线程能执行，如 `ReentrantLock`。又可分为公平锁和非公平锁：

- `Share（共享）`：多个线程可同时执行，如 `CountDownLatch`、`Semaphore`、 `CyclicBarrier`、`ReadWriteLock`

`ReentrantReadWriteLock` 可以看成是组合式，因为 `ReentrantReadWriteLock` 也就是读写锁允许多个线程同时对某一资源进行读

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 `state` 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），`AQS` 已经在顶层实现好了


#### AQS相关组件介绍

- **`ReentrantLock`（可重入锁）**

`ReentrantLock` 的 `state` 初始化为 0，表示未锁定状态。A 线程 `lock()` 时，会调用 `tryAcquire()` 独占该锁并将 `state + 1`（CAS实现）。此后，
其他线程再 `tryAcquire()` 时就会失败（失败后等待/重试由AQS维护），直到 `A` 线程 `unlock()` 到 `state = 0`（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A 线程自己是可以重复获取此锁的（state 会累加），这就是 **可重入的概念**。但要注意，获取多少次就要释放多么次，这样才能保证 `state` 是能回到0


- **`CountDownLatch`（倒计时器）**


`CountDownLatch` 任务分为 `N` 个子线程去执行，`state` 也初始化为 `N`（注意 `N` 要与线程个数一致）。这 `N` 个子线程是并行执行的，每个子线程执行完后 `countDown()` 一次，`state` 会CAS(Compare and Swap)减 `1`。等到所有子线程都执行完后(即 `state = 0` )，会 `unpark()` 主调用线程，然后主调用线程就会从 `await()` 函数返回，继续后余动作


- **`CyclicBarrier`（循环栅栏）**

`CyclicBarrier` 和 `CountDownLatch` 非常类似，它也可以实现线程间的技术等待，但是它的功能比 `CountDownLatch` 更加复杂和强大。主要应用场景和 `CountDownLatch` 类似。
`CyclicBarrier` 的字面意思是可循环使用（Cyclic）的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会开门，所有被屏障拦截的线程才会继续干活。`CyclicBarrier` 默认的构造方法是 `CyclicBarrier(int parties)`，其参数表示屏障拦截的线程数量，每个线程调用 `await()` 方法告诉 `CyclicBarrier` 我已经到达了屏障，然后当前线程被阻塞
