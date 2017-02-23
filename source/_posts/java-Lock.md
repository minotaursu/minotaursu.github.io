title: 'java Lock'
date: 2014-11-03 07:57:08
tags:
- java
- 锁
- 并发

---

在jdk5中，新增了Lock接口来提供一种比synchronized更灵活的锁机制。
## ReentrantLock

ReentrantLock是被最广泛使用来替代synchronized的Lock接口实现，ReentrantLock的代码使用非常简单，只要记得将unlock写在finally方法中就可以了。  
```
Lock lock = new ReentrantLock();
lock.lock();
try {
} finally {
	lock.unlock();
}
```

相比synchronied，ReentrantLock具有以下特性：
*  非阻塞的tryLock，可以进行锁的轮询。
*  能被中断的lockInterruptibly，可以进行锁中断。
*  有超时时间的tryLock

ReentrantLock的默认构造函数是非公平的，公平的锁机制会对锁的请求进行排队，按照FIFO的原则进行锁的获取，造成JVM对于等待中的线程调度次序和操作系统对线程的调度不匹配，所以公平锁的性能要小于非公平锁的性能；在大多数场景下我们都可以使用ReentrantLock的默认构造函数，而在某些需要避免‘饥饿’发生的场景下，我们可以使用ReentrantLock(true)。

## ReentrantReadWriteLock

ReentrantReadWriteLock实现了ReadWriteLock接口，提供了一对锁：readLock与writeLock。读锁可以由多个线程同时保持，读锁则是独占的。比较适合读多写少高并发的场景，相比于ReentrantLock，ReentrantReadWriteLock的使用场景并不广泛，性能也要差一点。

ReadWriteLock的互斥性：
* 读读不互斥
* 读写互斥
* 写写互斥

