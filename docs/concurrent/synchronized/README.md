### synchronized关键字


#### 讲一下synchronized关键字的底层原理

**synchronized 关键字底层原理属于 JVM 层面**，可以作用在 `实例方法`、`静态方法`、`代码块` 上。作用于实例方法时，锁对象为 **实例对象**；作用于静态方法时，锁对象为 **类对象**；作用于代码块时，可以指定锁对象。

- **`synchronized 同步语句块的情况`**

`synchronized` **同步语句块** 的实现使用的是 `monitorenter` 和 `monitorexit` 指令，其中 `monitorenter` 指令指向同步代码块的开始位置，`monitorexit` 
指令则指明同步代码块的结束位置，当执行 `monitorenter` 指令时，线程试图获取锁也就是获取 **对象监视器 monitor** 的持有权

>在 Java 虚拟机(HotSpot)中，Monitor 是基于 C++实现的，由ObjectMonitor实现的。每个对象中都内置了一个 ObjectMonitor对象。
>
>另外，wait/notify等方法也依赖于monitor对象，这就是为什么只有在同步的块或者方法中才能调用wait/notify等方法，否则会抛出java.lang.IllegalMonitorStateException的异常的原因

在执行 `monitorenter` 时，会尝试获取对象的锁，如果锁的计数器为 0 则表示锁可以被获取，获取后将锁计数器设为 1 也就是加 1。

在执行 `monitorexit` 指令后，将锁计数器设为 0，表明锁被释放。如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止

- **`synchronized 同步方法的情况`**

`synchronized` 修饰的方法并没有 `monitorenter` 指令和 `monitorexit` 指令，取得代之的确实是 `ACC_SYNCHRONIZED` 标识，该标识指明了该方法是一个同步方法。`JVM` 通过该 `ACC_SYNCHRONIZED` 访问标志来辨别一个方法是否声明为同步方法，从而执行相应的同步调用


**不管代码块还是同步方法两者的本质都是对对象监视器 monitor 的获取**


#### 谈谈synchronized和ReentrantLock的区别

- **两者都是可重入锁**

>“可重入锁” 指的是自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。同一个线程每次获取锁，锁的计数器都自增 1，所以要等到锁的计数器下降为 0 时才能释放锁

- **synchronized 依赖于 JVM 而 ReentrantLock 依赖于 API**

  - `synchronized` 是 `JVM` 层面实现的，并没有直接暴露给我们，会自动的获取和释放锁。

  - `ReentrantLock` 是 `JDK` 层面实现的（也就是 API 层面，需要手动 lock() 和 unlock() 方法配合 try/finally 语句块来完成锁的获取和释放），所以我们可以通过查看它的源代码，来看它是如何实现的

- **ReentrantLock 比 synchronized 增加了一些高级功能**

  - **等待可中断** : `ReentrantLock` 提供了一种能够中断等待锁的线程的机制，通过 `lock.lockInterruptibly()` 来实现这个机制。也就是说正在等待的线程可以选择放弃等待，改为处理其他事情。

  - **可实现公平锁** : `ReentrantLock` 可以指定是公平锁还是非公平锁。而 `synchronized` 只能是非公平锁。所谓的公平锁就是先等待的线程先获得锁。`ReentrantLock` 默认情况是非公平的，可以通过 `ReentrantLock` 类的 `ReentrantLock(boolean fair)` 构造方法来制定是否是公平的。

  - **可实现选择性通知（锁可以绑定多个条件）**: `synchronized` 关键字与 `wait()` 和 `notify()/notifyAll()` 方法相结合可以实现等待/通知机制。`ReentrantLock` 类当然也可以实现，但是需要借助于 `Condition` 接口与 `newCondition()` 方法。

>Condition是 JDK1.5 之后才有的，它具有很好的灵活性，比如可以实现多路通知功能也就是在一个Lock对象中可以创建多个Condition实例（即对象监视器），线程对象可以注册在指定的Condition中，从而可以有选择性的进行线程通知，在调度线程上更加灵活。 在使用notify()/notifyAll()方法进行通知时，被通知的线程是由 JVM 选择的，用ReentrantLock类结合Condition实例可以实现“选择性通知” 


#### synchronized锁升级过程

**`1.6之前重量级锁实现`**

![markdown](https://ddmcc-1255635056.file.myqcloud.com/d4bfd6ec-9902-460d-bd58-39298ae52868.png)

`synchronized` 底层由 `ObjectMonitor`（对象监视器） 实现的。每个对象中都内置了一个 `ObjectMonitor` 对象。当线程试图获取锁时也就是获取对象监视器 `monitor` 的持有权。 **`重量级锁`** 获取过程：

- 有二个线程A、线程B，要进行操作的时候 ，发现方法上加了 `synchronized` 锁，这时线程调度到A线程执行，A线程就抢先拿到了锁。拿到锁的步骤为：

  - 将 `MonitorObject` 中的持有者（_owner）设置成 A线程，并将进入次数（_count） + 1

  - 将 `mark word` 设置为 `Monitor` 对象地址，锁标志位改为10；

  - 将B线程阻塞放到 `ContentionList` 队列

- JVM 每次从等待的尾部取出一个线程放到 `OnDeck` 作为候选者，但是如果并发比较高，等待队列会被大量线程执行CAS操作，为了降低对尾部元素的竞争，将等待队列拆分成 `ContentionList` 和 `EntryList` 二个队列, JVM将一部分线程移到 `EntryList` 作为准备进 `OnDeck` 的预备线程

- 作为 `_owner` 的A 线程执行过程中，可能调用 `wait` 释放锁，这个时候A线程进入 `Wait Set` , 等待被唤醒

**`引入便向锁和轻量级锁后锁升级过程`**

- `无锁状态到偏向锁`

![markdown](https://ddmcc-1255635056.file.myqcloud.com/e851e648-59ae-42b5-ac5a-5ea8f3979af9.png)

  - 首先A 线程访问同步代码块，使用CAS 操作将 Thread ID 放到 Mark Word 当中；

  - 如果CAS 成功，此时线程A 就获取了锁

  - 如果线程CAS 失败，证明有别的线程持有锁，例如上图的线程B 来CAS 就失败的，这个时候启动偏向锁撤销 （revoke bias）；

  - 锁撤销流程：

    - 让 A线程在全局安全点阻塞（类似于GC前线程在安全点阻塞） 
    - 遍历线程栈，查看是否有被锁对象的锁记录（ Lock Record），如果有Lock Record，需要修复锁记录和Markword，使其变成无锁状态。
    - 恢复A线程 - 将是否为偏向锁状态置为 0 ，开始进行轻量级加锁流程

- `便向锁到轻量级锁`

  - 线程在自己的栈桢中创建锁记录 `Lock Record`

  - 线程A 将 `Mark Word` 拷贝到线程栈的 `Lock Record` 中，这个位置叫 displayced hdr，如下图所示


![markdown](https://ddmcc-1255635056.file.myqcloud.com/5bb54bb2-3a9d-443d-b69d-f3819201daa3.png)

  - 将锁记录中的Owner指针指向加锁的对象（存放对象地址）

  - 将锁对象的对象头的MarkWord替换为指向锁记录的指针。这二步如下图所示：

![markdown](https://ddmcc-1255635056.file.myqcloud.com/6b6e4756-26f4-4b01-82fa-1175da18d67e.png)

  - 这时锁标志位变成 `00` ，表示轻量级锁


**轻量级锁什么时候会升级为重量级锁**

**当锁升级为轻量级锁之后，如果依然有新线程过来竞争锁，首先新线程会自旋尝试获取锁，尝试到一定次数（默认10次）依然没有拿到，锁就会升级成重量级锁**

>  一般来说，同步代码块内的代码应该很快就执行结束，这时候线程B 自旋一段时间是很容易拿到锁的，但是如果不巧，没拿到，自旋其实就是死循环，很耗CPU的，因此就直接转成重量级锁咯，这样就不用了线程一直自旋了。这就是锁膨胀的过程
