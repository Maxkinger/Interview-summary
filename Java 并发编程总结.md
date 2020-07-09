## Java 并发编程总结

#### 进程和线程

#### 线程的状态

#### Thread 相关的各个方法

* thread.sleep

  sleep 在休眠一段时间后自动唤醒，并且休眠时不会释放掉锁

* thread.join

  是对 object.wait 方法的封装，会让调用线程获取到该 thread 对象锁，然后调用 thread.wait 使当前线程阻塞。在 thread 运行结束以后，会在 exit() 中调用 notifyAll() 唤醒调用者线程。

  ```java
  public final synchronized void join(long millis) throws InterruptedException {
          long base = System.currentTimeMillis();
          long now = 0;
          if (millis < 0) {
              throw new IllegalArgumentException("timeout value is negative");
          }
          if (millis == 0) {
              while (isAlive()) {
                  wait(0);
              }
          } else {
              // 这里使用 while 循环 wait 是防止线程被 interrupt，导致提前被唤醒，上面的 while 同理
              while (isAlive()) {
                  long delay = millis - now;
                  if (delay <= 0) {
                      break;
                  }
                  wait(delay);
                  now = System.currentTimeMillis() - base;
              }
          }
      }
  ```

* Thread.yield 静态方法，出让当前线程的时间片使用权，让调度器进行下一轮调度。不会阻塞当前线程。

* object.wait 线程获取到 object 锁，wait 进入阻塞状态，释放掉锁资源

* object.notify 线程获取到 object 锁，通知被 object wait 阻塞的线程，然后释放掉锁资源。具体唤醒哪个线程是随机的，notifyAll 可以唤醒所有因为 wait 被阻塞的线程。

* thread.interrupt 仅仅改变线程地 interrupt 标志位，不会立即停止线程执行。如果线程处于阻塞状态 sleep/join/wait，那么会抛出 InterruptedException

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

| 锁的分类   | 描述                                                         |
| ---------- | ------------------------------------------------------------ |
| 悲观锁     | 悲观地认为程序并发量很大，很容易产生数据修改冲突。比如 sychronized |
| 乐观锁     | 乐观地认为程序并发量并不大，不容易产生数据冲突，比如 CAS     |
| 公平锁     | 多线程竞争锁时，线程按照顺序排队                             |
| 非公平锁   | 多线程竞争锁时，线程没有顺序                                 |
| 独占锁     | 只能由被单个线程占有，比如 synchronized、ReentrantLock       |
| 共享锁     | 能被多个线程同时占有                                         |
| 可重入锁   | 当该线程获取到锁后，下一个同步语句不需要重新获取这个锁，比如 sychronized |
| 非可重入锁 | 当该线程获取到锁后，下一个同步语句需要重新获取这个锁         |
| 自旋锁     | 线程在竞争获取锁，没有获取到就一直进行循环尝试获取           |



#### synchronized 关键字



#### volatile 关键字



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

getUnsafe() 得到的实例，只能被 BootStrapClassLoader 类加载器加载，而不能被 AppClassLoader 加载。所以不能直接在代码中使用 getUnsafe 来获取 Unsafe 实例，但是可以通过**反射**绕过此机制来创建 Unsafe。

**所有在 JAVA_HOME/jre/lib 中的类都会由 BootStrap 来加载**。

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


#### JUC 之 ThreadLocalRandom 

ThreadLocalRandom 是 JDK 7 在 JUC 包下新增的随机数生成器，它弥补了 Random 类在多线程下的缺陷。

Random 类根据种子生成随机数 ，所以在多线程情况下，每个线程有可能获取到相同种子，从而每个线程得到相同的随机数。

Random 类中使用了 AtomicLong 类型的原子性 seed 来避免这种情况，如下：

```java
protected int next(int bits) {
        long oldseed, nextseed;
        AtomicLong seed = this.seed;
        do {
            oldseed = seed.get();
            nextseed = (oldseed * multiplier + addend) & mask;
        } while (!seed.compareAndSet(oldseed, nextseed));
        return (int)(nextseed >>> (48 - bits));
    }
```

但是在并发量很大的情况下，会造成过多线程处于自旋重试状态，降低并发性能，因此有 ThreadLocalRandom 来解决这个问题。

```mermaid
classDiagram
	Random <| -- ThreadLocalRandom
	calss ThreadLocalRandom
	ThreadLocalRandom : -SEED: long
	ThreadLocalRandom : -PROBE:	long
	ThreadLocalRandom : -SECONDARY: long
	ThreadLocalRandom : instance: ThreadLocalRandom
	ThreadLocalRandom : -probeGenerator: AtomicInteger
	ThreadLocalRandom : -seeder: AtomicLong
	ThreadLocalRandom : +nextInt(bound:int): int
	ThreadLocalRandom : nextSeed(): long
	ThreadLocalRandom : +current(): ThreadLocalRandom
	ThreadLocalRandom : localInit(): void
	Thread <-- ThreadLocalRandom
	class Thread {
		threadLocalRandomSeed: long
		threadLocalRandomProbe: int
		threadLocalRandomSecondarySeed: int
		+currentThread(): Thread
	}

	

```

如图所示，ThreadLocalRandom 和 ThreadLocal 类似，本质上都是工具类。ThreadLocalRandom 可以看成一个饿汉式单例。ThreadLocalRandom 的 current() 静态方法如下：

```java
public static ThreadLocalRandom current() {
        if (UNSAFE.getInt(Thread.currentThread(), PROBE) == 0)
            localInit();
        return instance;
    }
```

在 localInit 方法中，初始化当前调用者线程的三个 threadLocalRandom 变量：

```java
static final void localInit() {
        int p = probeGenerator.addAndGet(PROBE_INCREMENT);
        int probe = (p == 0) ? 1 : p; // skip 0
        long seed = mix64(seeder.getAndAdd(SEEDER_INCREMENT));
        Thread t = Thread.currentThread();
        UNSAFE.putLong(t, SEED, seed);
        UNSAFE.putInt(t, PROBE, probe);
    }
```

这里的 probeGenerator 和 seeder 是原子类，因为在多个线程同时初始化的时候，必须保证**每个线程的种子不同**。

再看 nextInt(int bound) 方法：

```java
public int nextInt(int bound) {
        if (bound <= 0)
            throw new IllegalArgumentException(BadBound);
        int r = mix32(nextSeed());
        int m = bound - 1;
        if ((bound & m) == 0) // power of two
            r &= m;
        else { // reject over-represented candidates
            for (int u = r >>> 1;
                 u + m - (r = u % bound) < 0;
                 u = mix32(nextSeed()) >>> 1)
                ;
        }
        return r;
    }

final long nextSeed() {
        Thread t; long r; // read and update per-thread seed
        UNSAFE.putLong(t = Thread.currentThread(), SEED,
                       r = UNSAFE.getLong(t, SEED) + GAMMA);
        return r;
    }
```

这些方法都是与线程无关的通用算法，因为种子是保存在线程内部的，所以这样也是线程安全的。

总而言之，**ThreadLocalRandom 就是帮助每个线程生成不同种子放在各个线程内部，这样线程产生随机数的时候可以互不影响。**

#### JUC 之 Atomic 包

JUC 包中提供了一系列的原子性操作类，都是使用非阻塞的 CAS 算法来实现的。要注意，**CAS 算法只能满足原子性，不能满足可见性和有序性，所以在如 AtomicInteger 等类中，value 是加了 volatile 关键字的，用来满足可见性和有序性。**

以 AtomicLong 为例，

```java
public class AtomicLong extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 1927816293512124184L;
    // 获取 unsafe 实例
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    // 存放值 value 的偏移量
    private static final long valueOffset;
    // 是否 JVM 是否支持 Long 类型无锁 CAS
   	static final boolean VM_SUPPORTS_LONG_CAS = VMSupportsCS8();
    private static native boolean VMSupportsCS8();

    static {
        try {
            // 获取 value 在该类中的属性的偏移量
            valueOffset = unsafe.objectFieldOffset
                (AtomicLong.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }
    
    // 使用 volatile 关键字保证可见性和有序性
    private volatile long value;

    /**
     * Creates a new AtomicLong with the given initial value.
     *
     * @param initialValue the initial value
     */
    public AtomicLong(long initialValue) {
        value = initialValue;
    }

    /**
     * Creates a new AtomicLong with initial value {@code 0}.
     */
    public AtomicLong() {
    }
    ...
}
```

再看递增操作：

```java
    public final long incrementAndGet() {
        return unsafe.getAndAddLong(this, valueOffset, 1L) + 1L;
    }
```

Unsafe 中：

```java
public final long getAndAddLong(Object var1, long var2, long var4) {
        long var6;
        do {
            var6 = this.getLongVolatile(var1, var2);
        } while(!this.compareAndSwapLong(var1, var2, var6, var6 + var4));// compareAndSwapLong 是 native 方法
        return var6;
    }
```

上面的代码很好懂。

并发量较低的情况下，CAS 优于 synchronized 关键字。但是在并发量高的情况下，由于大量线程的自旋，会导致白白浪费 CPU 资源，因此 JDK 8 新增了一个 LongAdder 类来克服高并发情况下 AtomicLong 的缺点。

##### LongAdder

LongAdder 的基本思想是