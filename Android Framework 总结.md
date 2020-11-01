## Android Framework 总结

### 一、android 系统架构

![android-stack_2x](Android Framework 总结.assets/android-stack_2x-1604220506189.png)

和 JVM 相比，DVM 是专门为移动设备定制的，可以在系统中运行多个虚拟机实例，并且每一个应用都有自己的虚拟机实例。多个实例可以防止在虚拟机崩溃时所有程序都被关闭。

**Dalvik 和 ART 的区别？**

* DVM 每次运行时，字节码都需要通过 JIT 转化为机器码，效率较低。而 ART 在安装应用时就会进行一次预编译，Ahead Of Time，将字节码预先编译成机器码存储在本地，这样每次运行时就不用执行编译了，效率提高。
* ART 在每个版本都优化了 GC 机制

### 二、Android 系统启动

