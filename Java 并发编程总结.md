## Java 并发编程总结

#### 进程和线程

#### 线程的状态

#### Thread 相关的各个方法

* thread.sleep

  sleep 在休眠一段时间后自动唤醒，并且休眠时不会释放掉锁

* thread.join

  是对 object.wait 方法的封装，会让调用线程获取到该 thread 对象锁，然后调用 thread.wait 使当前线程阻塞。在 thread 运行结束以后，会在 exit() 中调用 notifyAll() 唤醒调用者线程

* Thread.yield 静态方法，出让当前线程的时间片使用权，让调度器进行下一轮调度。不会阻塞当前线程。

* object.wait 线程获取到 object 锁，wait 进入等待阻塞，释放掉锁资源

* object.notify 线程获取到 object 锁，通知被 object wait 阻塞的线程，然后释放掉锁资源。具体唤醒哪个线程是随机的，notifyAll 可以唤醒所有因为 wait 被阻塞的线程。

* 

#### ThreadLocal

ThreadLocal



#### CAS  机制

CAS 是一种无锁编程的并发实现。JUC 的 Atomic 包 和 Lock 锁底层均是该机制，1.6 以后 sychronize 关键字在升级为重量级锁之前也是使用 CAS。

CAS 需要三个基本操作数，变量的内存地址 v，旧的预期值 A， 要修改的新值 B，如 JUC Atomic 包中的各种原子类的实现，比如：

```java
 public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update); 
     // compareAndSwapInt 是 native 方法
    }
```

什么是unsafe呢？Java语言不像C，C++那样可以直接访问底层操作系统，但是JVM为我们提供了一个后门，这个后门就是unsafe。unsafe为我们提供了硬件级别的原子操作。Java 中的 **CAS 原子操作的实现就依靠 Unsafe 类**。

至于valueOffset对象，是通过unsafe.objectFieldOffset方法得到，所代表的是AtomicInteger对象value成员变量在内存中的偏移量。我们可以简单地把valueOffset理解为value变量的内存地址。

旧的期望值加了 volatile 关键字，得到期望值后，在某个线程要更改变量值之前，将期望值和当前值进行比较，如果不相等则更改失败，该线程进入自旋状态，不断循环尝试 CAS 操作，直到操作成功。

* ABA 问题

  某个变量值 A->B->A ，当多个线程同时修改该变量，某个线程阻塞的情况下，可能会出现错误。

  加版本号解决 ABA 问题。**AtomicStampedReference** 类实现了版本号的 CAS。





