### ReentrantLock类

#### 介绍一下ReentrantLock

![markdown](https://ddmcc-1255635056.file.myqcloud.com/ab5d34a6-b390-4cde-955c-977a8b6e0926.png)

`ReentrantLock` 是基于 `AQS` 实现的对 **资源的公平与非公平独占锁**。通过重写 `AQS` 的 `tryAcquire` 和 `tryRelease` 方法实现的 `lock` 和 `unlock`与管理状态 `state`。

#### ReentrantLock是怎么实现可重入的？

在 `AQS` 中维护了两个变量：一个是 `state` 计数，一个是 `exclusiveOwnerThread` 拥有锁线程 。在获取锁时，先判断如果当前 `state > 0` 则说明锁已被获取，则判断获取锁的线程是不是当前线程，如果是则返回获取成功，并将 `state + 1`
当退出方法时 `state - 1`，直至 `state = 0`，则释放锁的占用，并设置 `exclusiveOwnerThread = null`


#### 说说公平锁与非公平锁

**公平锁在获取锁的时候会先判断当前是否有等待的队列**。而非公平不会判断是否有等待队列，直接尝试获取锁。有点像插队的感觉

非公平锁会有更好的性能。当然，非公平锁让获取锁的时间变得更加不确定，可能会导致在阻塞队列中的线程长期处于饥饿状态
