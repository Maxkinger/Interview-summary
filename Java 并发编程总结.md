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

* thread.interrupt 仅仅改变线程地 interrupt 标志位，不会立即停止线程执行。如果线程处于阻塞状态，sleep/join/wait，那么会抛出 InterruptedException

* thread.isInterrupted 是否被中断，不会重置标志位

  ```java
  public boolean isInterrupted() {
     		// false 不重置标志位
          return isInterrupted(false);
      }
  ```

* Thread.inerrupted 调用者线程是否被中断，重置标志位

  ```java
  public static boolean interrupted() {
          return currentThread().isInterrupted(true);
      }
  ```

  

#### ThreadLocal

ThreadLocal



#### 各种锁的分类

| 锁的分类   | 描述 |
| ---------- | ---- |
| 悲观锁     |      |
| 乐观锁     |      |
| 公平锁     |      |
| 非公平锁   |      |
| 独占锁     |      |
| 共享锁     |      |
| 可重入锁   |      |
| 非可重入锁 |      |
| 自旋锁     |      |





#### CAS  机制

CAS 是一种无锁编程的并发实现。JUC 的 Atomic 包 和 Lock 锁底层均是该机制，1.6 以后 sychronize 关键字在升级为重量级锁之前也是使用 CAS。

CAS 需要三个基本操作数，变量的内存地址 v，旧的预期值 A， 要修改的新值 B，如 JUC Atomic 包中的各种原子类的实现，比如：

```java
 public final boolean compareAndSet(int expect, int update) {
        return unsafe.compareAndSwapInt(this, valueOffset, expect, update); 
     // compareAndSwapInt 是 native 方法
    }
```

旧的期望值加了 volatile 关键字，得到期望值后，在某个线程要更改变量值之前，将期望值和当前值进行比较，如果不相等则更改失败，该线程进入自旋状态，不断循环尝试 CAS 操作，直到操作成功。

##### ABA 问题

某个变量值 A->B->A ，当多个线程同时修改该变量，某个线程阻塞的情况下，可能会出现错误。

加版本号解决 ABA 问题。**AtomicStampedReference** 类实现了版本号的 CAS。

#####  场景

CAS 属于乐观锁，乐观地认为程序地并发量不大，一般不会造成数据修改冲突，适用于并发量不大的业务场景。

#### Unsafe 类

Java 语言不像 C，C++ 那样可以直接访问底层操作系统，但是 JVM 为我们提供了一个后门，这个后门就是unsafe。unsafe 为我们提供了硬件级别的原子操作。Java 中的 **CAS 原子操作的实现就依靠 Unsafe 类**。

上述的 valueOffset 对象，是通过 unsafe.objectFieldOffset 方法得到，所代表的是 AtomicInteger 对象 value 成员变量在内存中的偏移量。我们可以简单地把 valueOffset 理解为 value 变量的内存地址。

Unsafe 类不能直接用 new 创建，构造函数私有。

```java
@CallerSensitive
    public static Unsafe getUnsafe() {
        Class var0 = Reflection.getCallerClass();
        if (!VM.isSystemDomainLoader(var0.getClassLoader())) {
            throw new SecurityException("Unsafe");
        } else {
            return theUnsafe;
        }
    }
```

getUnsafe() 得到的实例，只能被 BootStrapClassLoader 类加载器加载，而不能被 AppClassLoader 加载。所以不能直接在代码中使用 getUnsafe 来获取 Unsafe 实例，但是可以通过反射绕过此机制来创建 Unsafe。

#### 伪共享

CPU 中的缓存行大小一般是 2 的幂次方字节数，从主存中加载数据到缓存行中时，是一次加载一整个缓存行的大小。所以缓存行中不只有单个变量而是多个变量。当多个线程同时修改这多个变量时，实际上，某一时刻也只有一个线程能操作缓存行，相比于将多个变量分别放入到不同的缓存行，效率更低。体现了程序运行的**局部性原则**。

##### 如何避免伪共享？

* JDK8 以前通过字节填充来避免伪共享。

  ```java
  public final static class FilledLong {
          public volatile long value = 0L;
          public long p1,p2,p3,p4p5,p6; // 填充
      }
  ```

* JDK8 提供了 sun.misc.Contended 注解

  ```java
  @sun.misc.Contended
  public final static class FilledLong {
  	public volatile long value = 0L;
  }
  ```

  可以修饰类，也可以修饰变量。

  比如 Thread 类中：

  ```java
   /** The current seed for a ThreadLocalRandom */
      @sun.misc.Contended("tlr")
      long threadLocalRandomSeed;
  
      /** Probe hash value; nonzero if threadLocalRandomSeed initialized */
      @sun.misc.Contended("tlr")
      int threadLocalRandomProbe;
  
      /** Secondary seed isolated from public ThreadLocalRandom sequence */
      @sun.misc.Contended("tlr")
      int threadLocalRandomSecondarySeed;
  ```

  





