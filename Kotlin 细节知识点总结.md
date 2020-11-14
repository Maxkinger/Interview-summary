## Kotlin 细节知识点总结

### 1、by 和 by lazy 的区别

by 代理例子如下：

```kotlin
class Example {
    var p: String by Delegate()
}

class Delegate {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$thisRef, thank you for delegating '${property.name}' to me!"
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        println("$value has been assigned to '${property.name}' in $thisRef.")
    }
}
```

通过 by 代理的属性，代理对象可以和原对象类型不一致。代理对象的工作就是用自己 get 和 set 方法去代理掉原对象的 get 和 set 方法。

by lazy 例子如下：

```kotlin
val lazyValue: String by lazy {
    println("computed!")
    "Hello"
}

fun main() {
    println(lazyValue)
    println(lazyValue)
}
```

lazy by 只可以代理 val 类型的对象，而且代理对象就是一个 Lazy 对象。by lazy 其实是将 lambda 传入并构造了一个 Lazy 对象。Lazy 对象类型定义如下：

```kotlin
public interface Lazy<out T> {
    public val value: T
    public fun isInitialized(): Boolean
}
```

可以看到，Lazy 对象内部的 value 是不可变的，Lazy 代理的对象类型 T 被指定为 out，即 T 只能读不能写，所以只能代理 val 类型的可读对象。原对象的 get 同样被代理到 Lazy 的 get 方法，而该 Lazy 的 get 方法在原对象第一次被访问时会调用传入的 lambda 来实例化得到一个原对象实例。后续再次访问都会得到该实例。

by lazy 可以用来实现单例，默认情况下是 DCL 单例。

by lazy 能做的，by 也同样能做。

