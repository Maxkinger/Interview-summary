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