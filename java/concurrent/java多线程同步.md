# java多线程同步

## volatile

**volatile**用于修饰变量，使得变量在多个线程间具有可见性，一个线程对变量的更改立即对其它线程可见。

## synchronized

**synchronized**关键字可以用于修饰普通方法和静态方法，也可以以代码块的方式使用。作用于普通方法时，对当前对象上锁；作用于静态方法时，对类的class对象上锁；使用代码块的方式，则对参数指定的对象上锁。

**synchronized**可以配和Object方法wait()、notify()和notifyAll()实现线程间的等待通知机制。

## Lock

Lock接口有lock和unlock方法，分别对应资源的占有和释放。当一个线程占有锁时，其它线程调用lock()方法会被阻塞，直到锁被释放。Lock的常用实现有ReentrantLock。

**ReentrantLock**配合Condition的使用也可以实现等待/通知机制。

## volatile和synchronized的比较

- synchronized可以保证数据的可见性和操作的原子性，volatile只能保证数据的可见性。
- volatile性能比synchronized好，且volatile不会阻塞线程，synchronized可能会导致线程的阻塞。

## synchronized和ReentranLock的比较

- 底层实现不同。synchronized是jvm层面的实现，而ReentranLock是jdk层面的实现。
- 两者都是可重入锁。
- sychronized是非公平锁，ReentranLock可以配置为公平或非公平锁。
- ReentranLock使用Condition，可以实现更灵活的条件等待/通知。
