# AbstractQueuedSynchronizer

​	这里以ReentrantLock为例，通过Lock()和unlock() 观察AQS的整个生命周期

## 1.  lock() 方法是个入口

* sync.lock();
  * 非公平锁直接去抢占资源，成功则获取该锁(`compareAndSetState()`)，失败则进入下一步获取锁的过程
  * acquire(1)
    * tryAcquire()尝试直接去获取资源，如果成功则直接返回；
      * 如果当前状态是0，直接去抢占该资源
      * 如果当前线程是之前的独占线程，则状态+1（可重入）
    * addWaiter()将该线程加入等待队列的尾部，并标记为独占模式；
      * 成功则返回当前节点
      * 如果失败，自旋不断尝试将当前节点加入队列尾部
    * acquireQueued()使线程在等待队列中获取资源，一直获取到资源后才返回。如果在整个等待过程中被中断过，则返回true，否则返回false。
      * 自旋获取资源，如果前驱节点是head，尝试获取资源
      * 如果不是进入waitting状态，等待被unpark或interrupt
        * shouldParkAfterFailedAcquire
          * 拿到前驱的状态，如果是-1，已经确定会被通知，返回true，
          * 如果>0 ，前驱线程是无效状态，继续往前找，直到找到正常状态，并排在他后面
          * 如果前驱正常，那就把前驱的状态设置成-1，告诉它释放资源后通知自己一下。
        * parkAndCheckInterrupt
          * 调用park()使线程进入waiting状态
          * 如果被唤醒，查看自己是不是被中断的。
    * 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。

## 2. unlock() 释放锁

* sync.release(1);
  * tryRelease(arg) 尝试去释放指定量的资源
    * 如果当前线程不是独占锁的线程，抛异常
    * 如果state=0，则置空独占线程，返回true，并唤醒等待队列里的下一个线程
  * unparkSuccessor(h) 唤醒等待队列里的下一个线程
    * 置零当前线程所在的结点状态，允许失败
    * 找到下一个需要唤醒的结点
    * unpark 唤醒