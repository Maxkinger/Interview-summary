## 源码解析之 -- LiveData 深入研究

使用 LiveData 已经有很久了，也看过其中的源码，但是一直没有很好的总结。之前面试的时候也被问到 LiveData 的原理。现在梳理总结一下。

### 一、观察者模式

LiveData 是典型的观察者模式的应用。为了更好地理解 LiveData，先看一下观察者模式。

观察者模式包含**观察者**和**被观察者**，一般的结构如下：

<img src="/Users/bullfrog/Desktop/Interview-summary/文章集合/fig/Snipaste_2020-11-18_00-09-59.png" alt="观察者模式" style="zoom:50%;" />

ConcreteObersver 是具体的观察者，ConcreteObservable 是具体的被观察者。它们分别实现了 Observer 和 Observable 接口，这两个接口中定义了观察者和被观察者的行为。

被观察者 Obervable 一般拥有如下三个方法：

* `register(observer: Observer)`

  注册观察者，使被观察者知道自己要通知的对象。

* `unregister(observer: Observer)`

  注销观察者，后续不再通知。

* `notify()`

  通知观察者，让其做出响应。notify 方法常见实现如下：

  ```kotlin
  // 一个观察者
  fun notify() {
    observer.update()
  }
  // 多个观察者
  fun notify() {
    for (observer in List<Observer>) {
      observer.update()
    }
  }
  ```

  可以看到，观察者之所以能够**观察**到被观察者的变化，然后做出响应，本质上还是在于**被观察者在事件发生时主动地通知(notify)了观察者，并且调用了观察者的响应(update)方法**。（记得初学 Android 时觉得观察者模式很神奇，可以随时响应变化。实际上搞懂了以后，实现原理就是普通调用，其中更多的是一种思想）

观察者 Observer 拥有如下方法：

* `update()`

  当被观察者通知到观察者时，观察者调用该方法做出响应。

### 二、LiveData 结构

