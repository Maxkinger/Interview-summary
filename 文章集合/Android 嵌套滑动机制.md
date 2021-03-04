## Android 嵌套滑动机制

### 一、什么是嵌套滑动

嵌套滑动是 android 开发中常见的一种 UI 效果。当一个布局中包含多个可以滑动的 View，并且这些 View 互相嵌套的时候，就需要做嵌套滑动的处理来让 UI 交互有更流畅的效果。常见的比如 ScrollView 嵌套 RecyclerView：

![20200405191210528](C:%5CUsers%5Czxgu%5CDesktop%5C20200405191210528.gif)

如上所示，最外层的父布局可以滑动，内层的 RecyclerView 也可以滑动。当上滑 RecyclerView 的时候，最外层的父布局先上滑，直到上滑到 tab 的时候，这时候 RecyclerView 开始滑动，父布局停止滑动，并且手指不需要离开屏幕，可以一次性完成整个操作。这样就达到了内外布局连贯滑动的效果，并且达到了 tab 吸顶的效果。

### 二、滑动嵌套解决方案

那么如何才能做到这样的连贯的嵌套滑动呢？

#### 1、手动覆写事件分发与拦截

大家都很熟悉 android 的事件分发机制，要做到嵌套滑动的效果，重写事件分发是最为原始的一种方法。以之前展示的效果的为例，在 ACTION_MOVE 事件分发时，先判断 tab 位置是否到顶端，如果没到，则让外层父布局拦截掉 MOVE 事件，父布局滑动。如果已经到了，则不拦截，将事件传给子 RecyclerView，流程如下：

![image-20210304134358023](Android%20%E5%B5%8C%E5%A5%97%E6%BB%91%E5%8A%A8%E6%9C%BA%E5%88%B6.assets/image-20210304134358023.png)

是否拦截事件，即重写 `onIntercetTouchEvent` 方法，是该流程的核心。

##### 手动重写事件分发的缺点

* 只适合比较简单的嵌套滑动情况

  这一点很好理解。因为你需要自己手动编写拦截逻辑，嵌套滑动的布局一旦复杂，那么就需要大量的代码和逻辑来实现嵌套滑动，增大维护的成本。所以不适合复杂的嵌套滑动布局，也很难实现复杂的嵌套滑动，比如多级嵌套滑动。

* 难以支持 fling

  fling 指的是滑动松手后，视图继续依靠惯性滑动的过程。一般来说，考虑到用户体验，嵌套滑动是需要支持 fling 的。那么对于手动编写事件分发来说，除了需要重写 `onInterceptTouchEvent` 之外，还需要针对 `ACTION_UP` 事件进行特定的处理，因为 fling 源于 `ACTION_UP` 事件时产生的手指惯性。然而事件分发机制并没有提供像 `onInterceptTouchEvent` 的那样的对外暴露的接口让开发者来处理 `ACITON_UP` 事件。只能通过复写 `onTouchEvent` 等方法来处理，而这样做的限制太大，因为你需要调用 `super.onTouchEvent`，但是你又不能修改其中的代码。

* 没有办法实现连贯的吸顶嵌套滑动

  还是以之前的例子来说，当 tab 吸顶后，我们希望的是手指不松开继续往上滑可以使 RecyclerView 往上滑，然而手动拦截事件的做法是做不到的，必须先抬起手指然后再次滑动。为什么会这样？看看  `dispatchTouchEvent`  中的代码：

  ```java
  if (actionMasked == MotionEvent.ACTION_DOWN
                      || mFirstTouchTarget != null) {
                  ......
              } else {
                  // There are no touch targets and this action is not an initial down
                  // so this view group continues to intercept touches.
                  intercepted = true;
              }
  ```

  当 ViewGroup 在分发事件时，如果 `mFirstTouchTarget == null` 则说明 ViewGroup 中没有子 View 来消费事件，该事件由 ViewGroup 自己处理。而当 ViewGroup 拦截事件后，恰恰会将 `mFirstTouchTarget` 置空。回到之前的例子，当外层滑动父布局拦截了 `ACTION_MOVE` 事件后，会将 `mFirstTouchTarget` 置空。接下来即使吸顶后不拦截事件，由于 `mFirstTouchTarget` 已经为 null，所以事件不会传递到子 RecyclerView，而是继续由父布局消费。这样就没有达到连贯的吸顶嵌套滑动的效果。

  #### 2、CoordinatorLayout + AppBar + Behavior

  `CoordinatorLayout` 是 google 提供的一套可以实现复杂交互效果的布局，和 `AppBar` 、`Behavior` 配合使用，可以解耦地定制多种效果，这些效果由 `Behavior` 指定。并且 `Behavior` 可以自定义。

  要用 `CoordinatorLayout` 实现嵌套滑动非常简单，只要按如下编写布局文件：

  ```xml
  <androidx.coordinatorlayout.widget.CoordinatorLayout
      xmlns:android="http://schemas.android.com/apk/res/android"
      xmlns:app="http://schemas.android.com/apk/res-auto"
      xmlns:tools="http://schemas.android.com/tools"
      android:layout_height="match_parent"
      android:layout_width="match_parent">
  
      <com.google.android.material.appbar.AppBarLayout
          android:layout_width="match_parent"
          android:layout_height="wrap_content">
        ...
          <Button/>
        ...
      </com.google.android.material.appbar.AppBarLayout>
  
          <RecyclerView
              android:layout_width="match_parent"
              android:layout_height="wrap_content"
              app:layout_behavior="@string/appbar_scrolling_view_behavior"/>
  
</androidx.coordinatorlayout.widget.CoordinatorLayout>
  ```

  `AppBarLayout` 就是嵌套滑动中父布局里需要上滑的顶部，在 RecyclerView 中指定 `behavior` 为 `appbar_scrolling_view_behavior` 就可以实现最简单的嵌套滑动，如下：
  
  ！！！！！！！！！！！！！！！！
  
  插入 gif 图
  
  ！！！！！！！！！！！！！！！！
  
  看起来像带有 header 的 RecyclerView 在滑动，但其实是嵌套滑动。当然，如果要达到吸顶效果，只需要将顶部 tab 的
  
  的 button 属性添加 `app:layout_scrollFlags="scroll||enterAlwaysCollapsed"` 或 `app:layout_scrollFlags="scroll||enterAlwaysCollapsed"` 即可，效果如下：
  
  ！！！！！！！！！！！！！！！！
  
  插入 gif 图
  
  ！！！！！！！！！！！！！！！！
  
  `layout_scrollFlags` 和 `layout_behavior` 有很多可选值，配合起来可以实现多种效果，不只限于嵌套滑动。具体可以参考 API 文档。
  
  使用 `CoordinatorLayout` 实现嵌套滑动比手动实现要好得多，既可以实现连贯的吸顶嵌套滑动，又支持 fling。而且是官方提供的布局，可以放心使用，出 bug 的几率很小，性能也不会有问题。不过也正是因为官方将其封装得很好，使用 `CoordinatorLayout` 很难实现比较复杂的嵌套滑动布局，比如多级嵌套滑动。
  
  **注意**
  
  使用 `CoordinatorLayout` 有一个地方需要注意，当它的子 View 是 RecyclerView 或者 ScrollView 这种 content 可以无限长的布局时，要注意限制这些子 View 的高度，不要使用 `wrap_content` 设置子 View 的高度，因为 `CoordinatorLayout` 在测量时给子 View 的限制是 `UNSPECIFIED`  ，即不做限制。像 RecyclerView 如果内部 item 数量太多，RecyclerView 在 `wrap_content` 的情况下会把所有 item 都显示出来，相当于没有回收。这样会对内存造成很大消耗，如果调用 `setVisibility` 改变可见性的话，当从不可见到可见，更是会瞬间调用所有 item 的测量布局流程，造成卡顿。负一屏之前新闻列表开关的卡顿就是这个原因造成的。
  
  #### 3、嵌套滑动组件 NestedScrollingParent 和 NestedScrollingChild
  
  
  
  