## Flutter 学习疑问

### 1、Dart 不支持反射，那意味着没有运行时的注解？

### 2、Dart Mixin

Mixin 是一种可以将自己的方法借给其他类使用的类，而不用被其余类继承。在 Dart 中使用 `with` 关键字就可以做到这一点。

```dart
class B {  //B 不继承 A
  method(){
     ....
  }
}

class A with B {
  ....
     ......
}
void main() {
  A a = A();
  a.method();  // 在 A 中调用 B 的方法
}
```

或者：

```dart
class B {  //B is not allowed to extend any other class other than object
  method(){
     ....
  }
}

class C {
  ...
    ...
}

class A extends C with B { . //we can extends of implement class A
  ....
     ......
}
void main() {
  A a = A();
  assert(a is B);  // a is b = true
  assert(a is C);  // a is c = true
  a.method();  //we got the method without inheriting B
}

```

mixin 为多继承提供了高度的灵活性，同时避免了 DDD 问题。

使用 on 关键字可以限制 mixin 的使用范围，比如：

```dart
class A {}

class B{}

mixin X on A{}

mixin Y on B {} 

class P extends A with X {} // 只有继承自 A 的类才能 with X
class Q extends B with Y {} // 只有继承自 B 的类才能 with Y
```

### 3、一个 widget 对应一个 BuildContext 实例，还是多个 widget 可以对应一个 BuildContext 实例？

### 4、flutter 中使用到底如何使用 static、const、final 等关键字？

### 5、dart 不支持匿名内部类，只支持 lambda。dart 只能自己声明类，继承或实现另一个类，如下：

```dart
abstract class Event {
  void run();
}

class _AnonymousEvent implements Event {
  _AnonymousEvent({Function run}): _run = run;
  final void Function() _run;

  @override
  void run() => _run();
}

// 使用
Event event = _AnonymousEvent(run: () {});

```

### 6、flutter 中三棵树：widget 树、element 树和 renderObject 树的关系？

| Widget                        | Element                        |                                                              |
| ----------------------------- | ------------------------------ | ------------------------------------------------------------ |
| LeafRenderObjectWidget        | LeafRenderObjectElement        | Widget树的叶子节点，用于没有子节点的widget，通常基础组件都属于这一类，如Image。 |
| SingleChildRenderObjectWidget | SingleChildRenderObjectElement | 包含一个子Widget，如：ConstrainedBox、DecoratedBox等         |
| MultiChildRenderObjectWidget  | MultiChildRenderObjectElement  | 包含多个子Widget，一般都有一个children参数，接受一个Widget数组。如Row、Column、Stack等 |

这个问题搞得我头晕。。。

### 7、flutter PointerEvent 事件冒泡机制，为什么是从下往上？这和 android 事件分发机制相反？
### 8、Flutter 的  ListView 怎么这么卡？

debug 模式下运行确实很卡，但是在 release 模式下运行就不卡了，tricky，应该是因为两者编译方式不同。

### 9、Dart 的单例模式

* dart 中的私有构造函数，只需要在构造函数名称前加 _。私有构造函数只对 library 私有，同一个 library 中的其他类仍然能够访问该构造函数。
* 单例模式的标准做法就是 factory + static 变量

```dart
class Singleton {
    // 私有构造函数
    Singleton._internal();
    // 保存单例
    static Singleton _singleton = Singleton._internal();
    // 工厂构造函数
    factory Singleton() => _singleton;
}
// 定义一个top-level（全局）变量，页面引入该文件后可以直接使用bus
var bus = Singleton();
```

### 10、Notification

* 通知冒泡和触摸事件冒泡类似，都是从子节点向上逐层传递。

  Flutter中很多地方使用了通知，如可滚动组件（Scrollable Widget）滑动时就会分发**滚动通知**（ScrollNotification），而 Scrollbar 正是通过监听 ScrollNotification 来确定滚动条位置的。

* Flutter 的 UI 框架实现中，除了在可滚动组件在滚动过程中会发出`ScrollNotification`之外，还有一些其它的通知，如`SizeChangedLayoutNotification`、`KeepAliveNotification` 、`LayoutChangedNotification`等，Flutter正是通过这种通知机制来使父元素可以在一些特定时机来做一些事情。

### 11、Flutter 嵌套太多 widget 会不会造成性能问题？

参考：https://stackoverflow.com/questions/54193724/is-it-bad-to-have-a-lot-of-nested-widgets-with-flutter

简而言之，不会，简单 widget 的深度嵌套是 flutter 所推荐的做法。

flutter 的设计理念和 android 相反，flutter 认为“组合优于继承”，因此每一个 widget 都设计得十分轻量，方便快速构建。如果一个 widget 需要好几个功能，那么就组合多个轻量 widget 达到目的，而不是继承自某个 widget 再扩展功能。而 android 恰恰相反，它赋予了 View 这个最基本得视图类太多的东西，导致 View 的代码十分庞大。并且其余的控件全部要继承自 View，继承多而组合少。

**疑问：Flutter 是如何在渲染嵌套繁多的 widget 树的时候保持高性能的呢？它的绘制流程是什么？**

### 12、Dart 中的 export、part

### 13、Android 有类似 flutter 中 json_model 的工具吗？

GsonFormat 可以从 json 文件直接生成 Model class

