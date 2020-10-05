## Kotlin 协程

### 一、CoroutineScope & CoroutineContext

https://stackoverflow.com/questions/54416840/kotlin-coroutines-scope-vs-coroutine-context

#### CoroutineScope 

* 一个协程必须在 scope 里运行
* scope 可以追踪到在其内部运行的协程
* 所有的协程都可以通过这个 scope 取消
* scope 可以捕获 uncaught exceptions
* scope 是一种可以将协程绑定在应用特定生命周期的东西，比如 viewModelScope 可以避免 viewModel 泄露

#### CoroutineContext

context 决定了协程运行在哪个线程上。

* Dispatchers.Default - 适合计算密集型工作
* Dispatchers.Main - 主线程
* Dispatchers.Unconfined - 任意一个线程，没有限制
* Dispatchers.IO - 适合 IO 工作，比如数据库操作

![CoroutineContext](fig/image-20201005185242917.png)

  

 <img src="fig/image-20201005193143454.png" alt="image-20201005193143454" style="zoom:50%;" />