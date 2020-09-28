### 一、View 的绘制流程

#### onMeasure 

### 二、ANR 

一般来说，当主线程在5秒内没有对触摸事件进行响应时，就会产生 ANR。

但是，如果直接在 Activity 中 Thread.sleep(5000L)，会不会产生 ANR？我测试的结果，5000L 是不会 ANR 的，然而 6000L 就会出现 ANR。

### 三、OkHttp

#### 1、dispatcher

```java
public synchronized ExecutorService executorService() {
    if (executorService == null) {
      executorService = new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60, TimeUnit.SECONDS,
          new SynchronousQueue<>(), Util.threadFactory("OkHttp Dispatcher", false));
    }
    return executorService;
  }
```

dispatcher 就是一个 cachedThreadPoolExecutor。同步队列，队列中至多只有一个任务，是按需创建线程的线程池。

#### 2、RealCall.enqueue 和 RealCall.execute

realCall.enqueue -> dispatcher.enqueue -> dispatcher.promoteAndExecute -> 将 readyAsyncCalls 队列中的异步 call 转移到 runningAsyncCalls，然后挨个调用 AsyncCall.executeOn(dispatcher)，在此处调用线程池 dispatcher。

AsyncCall 是 RealCall 的内部类，这意味着它可以拿到构造它的 RealCall 的属性。同时 AsyncCall 继承自 Runnable，dispatcher 运行其 run 方法。

AsyncCall 的 run 方法：

```java
public final void run() {
    String oldName = Thread.currentThread().getName();
    Thread.currentThread().setName(name);
    try {
      execute();
    } finally {
      Thread.currentThread().setName(oldName);
    }
  }

protected void execute() {
      boolean signalledCallback = false;
      transmitter.timeoutEnter();
      try {
        Response response = getResponseWithInterceptorChain(); // (1)注意这里
        signalledCallback = true;
        responseCallback.onResponse(RealCall.this, response);
      } catch (IOException e) {
        if (signalledCallback) {
          // Do not signal the callback twice!
          Platform.get().log(INFO, "Callback failure for " + toLoggableString(), e);
        } else {
          responseCallback.onFailure(RealCall.this, e);
        }
      } catch (Throwable t) {
        cancel();
        if (!signalledCallback) {
          IOException canceledException = new IOException("canceled due to " + t);
          canceledException.addSuppressed(t);
          responseCallback.onFailure(RealCall.this, canceledException);
        }
        throw t;
      } finally {
        client.dispatcher().finished(this);
      }
    }
  }
```

代码 (1) 处调用 RealCall 的 getResponseWithInterceptorChain() 就是整个请求的核心了。

再看 realCall.execute

```java
public Response execute() throws IOException {
    synchronized (this) {
      if (executed) throw new IllegalStateException("Already Executed");
      executed = true;
    }
    transmitter.timeoutEnter();
    transmitter.callStart();
    try {
      client.dispatcher().executed(this); // （1）
      return getResponseWithInterceptorChain(); // （2）
    } finally {
      client.dispatcher().finished(this);
    }
  }
synchronized void executed(RealCall call) {
    runningSyncCalls.add(call);
  }
```

(1) 处只是把当前 call 加入了 runningSyncCalls 中，同步运行请求队列。同样，(2) 处的getResponseWithInterceptorChain() 是整个请求的核心。

#### 3、带拦截器的请求

getResponseWithInterceptorChain() 代码如下：

```java
Response getResponseWithInterceptorChain() throws IOException {
    // Build a full stack of interceptors.
    List<Interceptor> interceptors = new ArrayList<>();
    interceptors.addAll(client.interceptors());
    interceptors.add(new RetryAndFollowUpInterceptor(client));
    interceptors.add(new BridgeInterceptor(client.cookieJar()));
    interceptors.add(new CacheInterceptor(client.internalCache()));
    interceptors.add(new ConnectInterceptor(client));
    if (!forWebSocket) {
      interceptors.addAll(client.networkInterceptors());
    }
    interceptors.add(new CallServerInterceptor(forWebSocket));

    Interceptor.Chain chain = new RealInterceptorChain(interceptors, transmitter, null, 0,
        originalRequest, this, client.connectTimeoutMillis(),
        client.readTimeoutMillis(), client.writeTimeoutMillis());

    boolean calledNoMoreExchanges = false;
    try {
      Response response = chain.proceed(originalRequest);
      if (transmitter.isCanceled()) {
        closeQuietly(response);
        throw new IOException("Canceled");
      }
      return response;
    } catch (IOException e) {
      calledNoMoreExchanges = true;
      throw transmitter.noMoreExchanges(e);
    } finally {
      if (!calledNoMoreExchanges) {
        transmitter.noMoreExchanges(null);
      }
    }
  }
```

这里使用了拦截器模式。



### 四、Retrofit

#### 1、动态代理

retrofit 底层还是 okhttp ，只是对其进行了更好的包装，将构造请求的流程使用动态代理来完成，而简化了代码编写，只需要写请求参数即可。

何为动态代理？实质上就是在程序运行阶段生成字节码并加载到 JVM。

动态代理的关键在于如下代码：

```java
public <T> T create(final Class<T> service) {
    validateServiceInterface(service);
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();
          private final Object[] emptyArgs = new Object[0];

          @Override public @Nullable Object invoke(Object proxy, Method method,
              @Nullable Object[] args) throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            return loadServiceMethod(method).invoke(args != null ? args : emptyArgs);
          }
        });
  }
```

其中，Proxy.newProxyInstance 就是动态生成的 service 接口的实现类。而 InvocationHandler 就负责方法的具体实现。

#### 2、retrofit 如何把 service 的配置转化为 okhttp 的 call？

```java
// ServiceMethod
static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
    RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);

    Type returnType = method.getGenericReturnType();
    if (Utils.hasUnresolvableType(returnType)) {
      throw methodError(method,
          "Method return type must not include a type variable or wildcard: %s", returnType);
    }
    if (returnType == void.class) {
      throw methodError(method, "Service methods cannot return void.");
    }

    return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory); //(1)
  }
```

如上所示，重点在代码 (1) 处构造的 HttpServiceMethod。在解析了 method 的各种注解之后，根据这些信息生成了一个 HttpServiceMethod。

```java
static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
    ...
    CallAdapter<ResponseT, ReturnT> callAdapter =
        createCallAdapter(retrofit, method, adapterType, annotations);
    ...
    return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
    ...
}
```

而 HttpServiceMethod 的 invoke 方法是这样的：

```java
@Override final @Nullable ReturnT invoke(Object[] args) {
    Call<ResponseT> call = new OkHttpCall<>(requestFactory, args, callFactory, responseConverter); // （1）
    return adapt(call, args); // (2)
  }
```

代码 (1) 处直接 new 了一个 OkHttpCall，将 ServiceMethod 信息封装在了 OkHttpCall 里。在需要的时候，可以使用该 OkHttpCall 中的信息来创建一个 okhttp3.Call 对象，使用这个对象发起网路请求。

代码 (2) 处则把 call 对象转化为需要的返回值类型，后台线程的请求得到的结果，回在响应后切回主线程执行回调。

总而言之，ServiceMethod 包含了请求方法的注解信息，而其 invoke 方法则生成一个 OkHttpCall 对象，这个对象可以用来向底层的 okhttp 发起请求。









