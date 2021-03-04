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

  #### 3、嵌套滑动组件 NestedScrollingParent 和 NestedScrollingChild

​    `NestedScrollingParent ` 和 `NestedScrollingChild` 是 google 官方提供地一套专门用来解决嵌套滑动地组件。它们是两个接口，如下：

```java
public interface NestedScrollingParent2 extends NestedScrollingParent {

    boolean onStartNestedScroll(@NonNull View child, @NonNull View target, @ScrollAxis int axes,
            @NestedScrollType int type);

    void onNestedScrollAccepted(@NonNull View child, @NonNull View target, @ScrollAxis int axes,
            @NestedScrollType int type);

    void onStopNestedScroll(@NonNull View target, @NestedScrollType int type);

    void onNestedScroll(@NonNull View target, int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @NestedScrollType int type);

    void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed,
            @NestedScrollType int type);

}

public interface NestedScrollingChild2 extends NestedScrollingChild {
    
    boolean startNestedScroll(@ScrollAxis int axes, @NestedScrollType int type);

    void stopNestedScroll(@NestedScrollType int type);

    boolean hasNestedScrollingParent(@NestedScrollType int type);

    boolean dispatchNestedScroll(int dxConsumed, int dyConsumed,
            int dxUnconsumed, int dyUnconsumed, @Nullable int[] offsetInWindow,
            @NestedScrollType int type);

    boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow, @NestedScrollType int type);
}

```

需要嵌套滑动的 View 可以实现这两个接口，复写其中的方法。这套组件实现嵌套滑动的核心原理很简单，主要是以下三步：

* child 在 `onTouchEvent` 方法中先将 `ACITON_MOVE` 事件产生的位移 dx 和 dy 通过 `dispatchNestedPreScroll` 传递给 parent
* parent 在 `onNestedPreScroll` 中接受到 dx 和 dy 并进行消费。并将消费掉的位移放入 `int[] consumed` 中，`consumed` 数组是一个长度为 2 的 int 类型数组，`consumed[0]` 代表 `x` 的消耗，`consumed[1]` 代表 `y` 的消耗
* child 之后从 `int[] consumed` 数组中拿到 parent 已经消费掉的数值，减去之后得到剩余的位移，再由自己消费

事件由 child -> parent -> child，如果 child 是 recyclerview ，它会先把位移给父布局消费，这时父布局滑动。当父布局滑动顶到不能滑动时，recyclerview 这时会消费全部位移，这时它自己开始滑动，这样就形成了嵌套滑动，正如之前的例子中所看到的。

![image-20210304154351096](Android%20%E5%B5%8C%E5%A5%97%E6%BB%91%E5%8A%A8%E6%9C%BA%E5%88%B6.assets/image-20210304154351096.png)

`dispatchNestedScroll` 和 `onNestedScroll` 的作用原理上述  preScroll 的方法类似，只不过这两个方法构造的嵌套滑动顺序和 preScroll 的相反，是子 View 先滑动，当子 View 到顶后，父 View 再滑动。

这套机制还支持 fling，在手指离开 view 的时候，即产生 `ACITON_UP` 事件时，child 将此时的 velocity 转化为位移 dx 和 dy，并重复之前的流程。通过 `@NestedScrollType int type` 的值来判断是 `TYPE_TOUCH` 还是 `TYPE_NON_TOUCH`。

##### Android 中哪些 View 使用了这套滑动机制？

* 实现 `NestedScrollingParent` 接口的 View 有：NestedScrollView、CoordinatorLayout、MotionLayout 等
* 实现 `NestedScrollingChild` 接口的 View 有：NestedScrollView、RecyclerView 等
* NestedScrollView 是唯一同时实现两个接口的 View，这意味着它可以用作中介来实现多级嵌套滑动，后面会说到。

从上面可以看到，实际上，之前提到的 CoordinatorLayout 实现的嵌套滑动，本质上也是通过这套 NestedScrolling 接口来实现的。但是由于它封装得太好，我们没办法做过多定制。而直接使用这套接口，就可以根据自己的需求做定制。

大部分的场景中，我们不需要去实现 NestedScrollingChild，因为 RecyclerView 已经做了这个实现，而涉及到嵌套滑动的子 View 基本也都是 RecyclerView。我们看看 RecyclerView 的相关源码：

```java
public boolean onTouchEvent(MotionEvent e) {
    ...
    case MotionEvent.ACTION_MOVE: {
               ...
                // 计算 dx，dy
                int dx = mLastTouchX - x;
                int dy = mLastTouchY - y;
				...
                    mReusableIntPair[0] = 0;
                    mReusableIntPair[1] = 0;
       				...
                    // 分发 preScroll
                    if (dispatchNestedPreScroll(
                        canScrollHorizontally ? dx : 0,
                        canScrollVertically ? dy : 0,
                        mReusableIntPair, mScrollOffset, TYPE_TOUCH
                    )) {
                        // 减去父 view 消费掉的位移
                        dx -= mReusableIntPair[0];
                        dy -= mReusableIntPair[1];
            
                        mNestedOffsets[0] += mScrollOffset[0];
                        mNestedOffsets[1] += mScrollOffset[1];
                        
                        getParent().requestDisallowInterceptTouchEvent(true);
                    }
        ...
            } break;
    ...
}

boolean scrollByInternal(int x, int y, MotionEvent ev) {
        int unconsumedX = 0;
        int unconsumedY = 0;
        int consumedX = 0;
        int consumedY = 0;
        if (mAdapter != null) {
            mReusableIntPair[0] = 0;
            mReusableIntPair[1] = 0;
            // 先消耗掉自己的 scroll
            scrollStep(x, y, mReusableIntPair);
            consumedX = mReusableIntPair[0];
            consumedY = mReusableIntPair[1];
            // 计算剩余的量
            unconsumedX = x - consumedX;
            unconsumedY = y - consumedY;
        }

        mReusableIntPair[0] = 0;
        mReusableIntPair[1] = 0;
    	// 分发 nestedScroll 给父 View，顺序和 preScroll 刚好相反
        dispatchNestedScroll(consumedX, consumedY, unconsumedX, unconsumedY, mScrollOffset,
                TYPE_TOUCH, mReusableIntPair);
        unconsumedX -= mReusableIntPair[0];
        unconsumedY -= mReusableIntPair[1];
		...
    }
```

RecyclerView 是怎么调到父 View 的 `onNestedSroll` 和 `onNestedScroll` 的呢？分析一下 `dispatchNestedPreScroll` 的代码，如下，
`dispatchNestedScroll` 的代码原理和此类似，不再贴出：

```java
public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow,
            int type) {
        return getScrollingChildHelper().dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow,
                type);
    }

// NestedScrollingChildHelper.java
public boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow, @NestedScrollType int type) {
        if (isNestedScrollingEnabled()) {
            final ViewParent parent = getNestedScrollingParentForType(type);
            if (parent == null) {
                return false;
            }

            if (dx != 0 || dy != 0) {
                ...
                consumed[0] = 0;
                consumed[1] = 0;
                ViewParentCompat.onNestedPreScroll(parent, mView, dx, dy, consumed, type);
				...
                return consumed[0] != 0 || consumed[1] != 0;
            } 
            ...
        }
        return false;
    }

// ViewCompat.java
public static void onNestedPreScroll(ViewParent parent, View target, int dx, int dy,
            int[] consumed, int type) {
        if (parent instanceof NestedScrollingParent2) {
            // First try the NestedScrollingParent2 API
            ((NestedScrollingParent2) parent).onNestedPreScroll(target, dx, dy, consumed, type);
        } else if (type == ViewCompat.TYPE_TOUCH) {
            // Else if the type is the default (touch), try the NestedScrollingParent API
            if (Build.VERSION.SDK_INT >= 21) {
                try {
                    parent.onNestedPreScroll(target, dx, dy, consumed);
                } catch (AbstractMethodError e) {
                    Log.e(TAG, "ViewParent " + parent + " does not implement interface "
                            + "method onNestedPreScroll", e);
                }
            } else if (parent instanceof NestedScrollingParent) {
                ((NestedScrollingParent) parent).onNestedPreScroll(target, dx, dy, consumed);
            }
        }
    }
```

可以看到，RecyclerView 通过一个代理类 `NestedScrollingChildHelper` 完成滑动分发，最后交给 `ViewCompat` 的静态方法来让父 View 处理 `onNestedPreScroll`。`ViewCompat` 的主要作用是用来兼容不同版本的滑动接口。

从上面的代码可以清楚地看到 RecyclerView 对于 `NestedScrollingChild` 的实现，以及触发嵌套滑动的时机。如果我们要实现嵌套滑动，并且内部的滑动子 View 是 RecyclerView，那么只需要将外层的父 View 实现 `NestedScrollingParent` 的方法就行了，比如在 `onNestedPreScroll`方法中，

```java
 @Override
    public void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed, int type) {
     	// 滑动 dy 距离
       scrollBy(0, dy);
        // 将消耗掉的 dy 放入 consumed 数组通知子 view
       consumed[1] = dy;
    }
```

另外，像 `NestedScrollView` 这样的类，它内部实现了 `onNestedScroll`，所以它能在内部的 RecyclerView 滑动到顶端时继续下滑而不用抬起手指。另外也实现了 `onNestedPreScroll`方法，只不过它在该方法中把滑动继续向上传递，自己没有消费，如下代码：

```java
// NestedScrollView.java
public void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed,
            int type) {
    // 只分发了 preScroll 自己并没有消费。之所以能分发是因为 NestedScrollView 同时实现了 NestedScrollingChild 接口
        dispatchNestedPreScroll(dx, dy, consumed, null, type);
    }

@Override
    public boolean dispatchNestedPreScroll(int dx, int dy, int[] consumed, int[] offsetInWindow,
            int type) {
        return mChildHelper.dispatchNestedPreScroll(dx, dy, consumed, offsetInWindow, type);
    }

// NestedScrollingChildHelper.java
public boolean dispatchNestedPreScroll(int dx, int dy, @Nullable int[] consumed,
            @Nullable int[] offsetInWindow, @NestedScrollType int type) {
        if (isNestedScrollingEnabled()) {
            final ViewParent parent = getNestedScrollingParentForType(type);
            if (parent == null) {
                return false;
            }

            if (dx != 0 || dy != 0) {
                ...
                consumed[0] = 0;
                consumed[1] = 0;
                ViewParentCompat.onNestedPreScroll(parent, mView, dx, dy, consumed, type);
				...
                return consumed[0] != 0 || consumed[1] != 0;
            } 
            ...
        }
        return false;
    }
```

**Tips**：`NestedScrollingParent` 和 `NestedScrollingChild` 一共有 3 个版本。最早的是 `NestedScrollingParent` 和 `NestedScrollingChild`，这一套接口把 scroll 和 fling 分别进行处理，造成了不必要的复杂性。后来有了 `NestedScrollingParent2` 和 `NestedScrollingChild2` 继承自一代，不过它将 fling 转化为 scroll 的距离统一进行处理。上述的嵌套滑动组件均指二代。后来又有了 `NestedScrollingParent3` 和 `NestedScrollingChild3`继承自二代，它们相比于 2 代增加了 `dispatchNestedScroll` 和 `onNestedScroll` 消耗部分滑动的功能。一般来说，要自己实现嵌套滑动，只需要实现 2 代及以上接口即可。一代基本不再使用。

 **注意**：使用 `NestedScrollView` 有一个地方需要注意，当它的子 View 是 RecyclerView 这种 content 可以无限长的布局时，要注意限制这些子 View 的高度，不要使用 `wrap_content` 设置 RecyclerView 的高度，因为 `NestedScrollView` 在测量时给子 View 的限制是 `UNSPECIFIED`  ，即不做限制，RecyclerView 想要多高就有多高。像 RecyclerView 如果内部 item 数量太多，RecyclerView 在 `wrap_content` 的情况下会把所有 item 都显示出来，相当于没有回收。这样会对内存造成很大消耗，如果调用 `setVisibility` 改变可见性的话，当从不可见到可见，更是会瞬间调用所有 item 的测量布局流程，造成卡顿。**负一屏之前新闻列表开关的卡顿就是这个原因造成的**。

 ### 三、多级嵌套滑动

我们知道了 `NestedScrollingParent` 和 `NestedScrollingChild`可以用来定制化地实现自己地嵌套滑动。很容易想到，如果一个 View 同时实现了两个接口，那么它既可以接受 child 的滑动，又可以分发滑动给 parent，这样就形成了一个链条。而多级嵌套滑动的核心原理就来自于此，如图：

![image-20210304170155409](Android%20%E5%B5%8C%E5%A5%97%E6%BB%91%E5%8A%A8%E6%9C%BA%E5%88%B6.assets/image-20210304170155409.png)

原理其实并不复杂，下面用伪代码表示一下：

* 对于 `NestedScrollingParent`

  ```
   @Override
      public void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed, int type) {
         scrollByMe(dx, dy);
         ...
         consumed[0] = dxConsumed;
         consumed[1] = dyConsumed;
      }
  ```

* 对于中介者，即同时实现了 `NestedScrollingParent` 和 `NestedScrollingChild` 的中间 View 来说

  ```java
   @Override
      public void onNestedPreScroll(@NonNull View target, int dx, int dy, @NonNull int[] consumed, int type) {
          // 先分发，再消费。当然也可以先以先消费，再分发，这取决于业务
         dispatchNestedPreScroll(dispatchNestedPreScroll(dx, dy, consumed, null, type);
  	   int dx -= consumed[0];
         int dy -= consumed[1];
         scrollByMe(dx, dy);
         consumed[0] = dxConsumed;
         consumed[1] = dyConsumed;
      }
  ```

* 对于最内层的 `NestedScrollingChild`，一般使用 RecyclerView 就可以

### 四、嵌套滑动组件中使用的设计模式

作为总结，讨论一下。

* 策略模式。

  `NestedScrollingParent` 和 `NestedScrollingChild`是一对接口，不同的 View 通过实现这对接口来达到不同的嵌套滑动效果。同时使用接口也保证了扩展性。

* 代理模式。

  如前述，当一个 View 实现嵌套滑动接口中的方法时，滑动的具体传递都交给了代理的 `NestedScrollingParentHelper` 和 `NestedScrollingChildHelper`来实现，这两个类是由 sdk 提供的，在 `NestedScrollingParent` 和 `NestedScrollingChild` 接口中有如下说明：

  ```java
  This interface should be implemented by ViewGroup subclasses
  that wish to support scrolling operations delegated by a nested child view.
  Classes implementing this interface should create a final instance of a
  NestedScrollingParentHelper as a field and delegate any View or ViewGroup methods
  to the NestedScrollingParentHelper methods of the same signature.
  ```

* 适配器模式 / 外观模式

  RecyclerView 实现了 `NestedScrollingChild2` 接口，但是如果它的父 view 实现的是 `NestedScrollingParent` 接口怎么办？这就不同版本的嵌套滑动组件需要兼容。怎样实现兼容呢，使用 `ViewCompat` ，如下：

  ```java
  // ViewCompat.java
  public static void onNestedPreScroll(ViewParent parent, View target, int dx, int dy,
              int[] consumed, int type) {
          if (parent instanceof NestedScrollingParent2) {
              // First try the NestedScrollingParent2 API
              ((NestedScrollingParent2) parent).onNestedPreScroll(target, dx, dy, consumed, type);
          } else if (type == ViewCompat.TYPE_TOUCH) {
              // Else if the type is the default (touch), try the NestedScrollingParent API
              if (Build.VERSION.SDK_INT >= 21) {
                  try {
                      parent.onNestedPreScroll(target, dx, dy, consumed);
                  } catch (AbstractMethodError e) {
                      Log.e(TAG, "ViewParent " + parent + " does not implement interface "
                              + "method onNestedPreScroll", e);
                  }
              } else if (parent instanceof NestedScrollingParent) {
                  ((NestedScrollingParent) parent).onNestedPreScroll(target, dx, dy, consumed);
              }
          }
      }
  ```

  所有的 child 的滑动分发，都会通过 `ViewCompat` 的静态方法最终再传递给 parent，通过这个类以及静态方法，达到了不同版本嵌套滑动组件的兼容。同时， `ViewCompat` 对外暴露了易用的接口，而把兼容的过程隐藏在了自己内部，也可以看成是一种外观模式。