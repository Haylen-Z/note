# **A**bstract**Q**ueued**S**ynchronizer

 **AQS**是java.util.concurrent.locks下的一个类，用来实现锁和同步器的一个框架。
 
 AQS内部有一个双向队列，每个Node代表一个正在等待的线程；AQS还有有个叫state的整型变量，用来控制线程的同步，state = 0表示没有线程获取锁，state > 0表示有线程占有锁。
 
 一个线程尝试获取锁，如果成功则占有锁，失败则会被作为一个节点添加到队列中，阻塞等待。
 
 占有锁的线程释放锁时，会唤醒队列中的线程。被唤醒的线程检查自己的前置节点是否为头节点，是则尝试获取锁，否则阻塞等待；如果尝试获取锁成功，则删去旧头节点并把自己设计为新的头节点，否则阻塞等待。
 
 利用AQS可以实现独占锁（ReentrantLock）和 共享锁（ReentrantReadWriteLock），AQS提供了几个模板方法供子类实现：
 
         isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
         tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
         tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
         tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
         tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。

其中的int参数为state。

 
 
