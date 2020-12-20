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

