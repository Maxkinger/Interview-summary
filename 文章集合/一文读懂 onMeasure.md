## 一文理解 onMeasure

记得我刚接触自定义 View 的时候，关于 View 的测量、布局、绘制三大流程，最难懂的就是 `onMeasure` 过程。相比于 `onLayout` 和 `onDraw` 只关注当前 View 的逻辑，`onMeasure` 往往要处理父 View 和子 View 之间关系，这让 `onMeasure` 理解和实践起来都变得更难了一些。当时并没有理解的很透彻，不过还是能做的。后来随着经验提高，慢慢搞清楚了 `onMeasure` 过程，并且发现网上很多关于 `onMeasure` 的一些说法都是错的，或者说不准确的。

### 一、MeasureSpec 来自哪里

我们看看 View 的 `onMeasure` 方法：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

我们都知道在自定义 View 的时候要重写 `onMeasure` 方法，要对方法的两个参数 `widthMeasureSpec` 和 `heightMeasureSpec` 进行处理。网上很多文章在提到 `measureSpec` 的时候会详细的说它的概念，比如 32 位整数，头两位是 mode，后面 30 位是 size。但是往往都是罗列一大串知识点，并没有理清其中的关系和逻辑。所以这篇文章我想换一种思路来讲 `onMeasure` ，尽量避免罗列知识点，而是从我自己的理解出发，写清楚 `onMeasure` 的逻辑，以及为什么这样做。

首先，`widthMeasure` 和 `heightMeasure` 来自哪里？是谁把这两个参数传入到自定义 View 的 `onMeasure` 方法里的？

要弄清这个问题，就要知道是谁调用了 View 的 `onMeasure` 方法。这个大家应该都很清楚了，在 View 的 `measure` 方法中会调用 `onMeasure`，如下：

```java
public final void measure(int widthMeasureSpec, int heightMeasureSpec) {
  	...
    onMeasure(widthMeasureSpec, heightMeasureSpec);
    ...
}
```

可以看到，`onMeasure` 方法中的 `measureSpec` 来自于 `measure` 方法中的 `measureSpec`，那么是谁调用了 View 的 `measure` 方法呢？答案是它的父 View。也就是说，调用 `measure` 方法的是 ViewGroup 类型。其实，在 `measure` 方法的注释中，也可以看出来：

> ```java
> This is called to find out how big a view should be. The parent
> supplies constraint information in the width and height parameters.
> @param widthMeasureSpec Horizontal space requirements as imposed by the
>        parent
> @param heightMeasureSpec Vertical space requirements as imposed by the
>        parent
> ```

`widthMeasureSpec` 和 `heightMeasureSpec` 是由父 View 和子 View 的尺寸共同作用得到的对子 View 施加的约束。

以 LinearLayout 为例，在其 `onMeasure` 方法中，会有如下调用链：

> `onMeasure` -> `measureVertical` -> `measureChildBeforeLayout` -> `measureChildWithMargins` 

在 `measureVertical` 方法中，会遍历其子 View，调用 `measureChildBeforeLayout` 方法

```java
final int count = getVirtualChildCount();
for (int i = 0; i < count; ++i) {
  ...
  measureChildBeforeLayout(child, i, widthMeasureSpec, 0,
                        heightMeasureSpec, usedHeight);
}
```

在 `measureChildBeforeLayout` 中会调用 `measureChildWithMargins` ，这个方法内部就调用了子 View 的 `measure` 方法，如下，这个方法其实是 ViewGroup 这个基类中的方法：

```java
// ViewGroup.java
protected void measureChildWithMargins(View child,
            int parentWidthMeasureSpec, int widthUsed,
            int parentHeightMeasureSpec, int heightUsed) {
        final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
				// 计算传递给子 View 的 measureSpec
        final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
        final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
                mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                        + heightUsed, lp.height);
				// 调用子 View 的 measure 方法
        child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

该方法的最后一句就是调用 LinearLayout 的子 View 的 `measure` 方法，并把计算得到的 `measureSpec` 传入该方法。这里要注意的是，虽然子 View 在 `onMeasure` 过程中的 `measureSpec` 是由父 View 传进来的，但是这个 `measureSpec` 本身是属于子 View 的，它内部包含的仅仅是子 View 的尺寸信息。只不过在得到这个子 View 尺寸信息的过程中，需要借助父 View。

### 二、MeasureSpec 如何计算

前面我们知道，子 View 在 `onMeasure` 过程使用的 `measureSpec` 是由父 View 给它的。那么父 View 是怎么计算得到这个 `measureSpec` 的呢？

在 `measureChildWithMargins` 方法中，有：

```java
final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                        + widthUsed, lp.width);
```

顾名思义，子 View 的 `measureSpec` 是由 `getChildMeasureSpec` 方法计算来的。该方法签名如下：

```java
public static int getChildMeasureSpec(int spec, int padding, int childDimension)
```

* `spec` ，这个参数指的是父 View 的 `measureSpec`，也就是调用子 View 的 `measure` 方法的那个父 View。当然，你可能会问，父 View 的 `measureSpec` 是从哪里来的？答案是父 View 的 父 View，这是一个链条。
* `padding`，这个参数是父 View 的 padding 和子 View  的 margin 等值的和，在这里不是重点。
*  `childDimension`，这个指的就是我们 View 的宽度或者高度。也就是我们在 xml 设置的 `android:layout_height` 或者 `android:layout_widht`。`childDimension` 可能是一个具体的值，比如 `24dp`（当然，在 `measure` 过程中已经转为了对应的 `px` ），也可以是 `MATCH_PARENT` 或者 `WRAP_CONTENT`，这两者对应的值分别为 -1 和 -2，属于标记值。

**注意**：网上的很多文章，在看的时候很容易让人模糊了 `MATCH_PARENT` 及 `WRAP_CONTENT`  这两者与 `MeasureSpec.MODE` 的关系。一些文章会说，当 View 的尺寸设置为 `WRAP_CONTENT` 的时候，它的 mode 对应 `AT_MOST`。这种说法并不准确。 `MATCH_PARENT` 和`WRAP_CONTENT` 只是 `childDimension`，它和 `measureSpec` 的 mode 是两个不同的概念，只不过它是标记用的 `childDimension`，需要借助父 View 给出具体的尺寸。

在深入 `getChildMeasureSpec` 的具体实现之前，我们回顾一下 `MeasureSpec` 的概念。

![image-20210512111013280](%E4%B8%80%E6%96%87%E8%AF%BB%E6%87%82%20onMeasure.assets/image-20210512111013280.png)

`MeasureSpec` 的结构对于开发者来说应该很熟悉了，是一个 32 位 `Int` 整型。高 2 位是 `MODE`，低 30 位是 `SIZE`。`getChildMeasureSpec` 的返回值就是一个  `MeasureSpec`，这个 `MeasureSpec` 最后会作为参数传入到子 View 的 `onMeasure` 方法中。

`MODE` 有 3 种：

* `EXACTLY`  

  表示当前 View 的尺寸为确切的值，这个值就是后 30 位 `SIZE` 的值。

* `AT_MOST`

  表示当前 View 的尺寸最大不能超过 `SIZE` 的值。

* `UNSPECIFIED`

  表示当前 View 的尺寸不受父 View 的限制，想要多大就可以多大。这种情况下，`SIZE` 的值意义不大。一般来说，可滑动的父布局对子 View 施加的约束就是 `UNSPECIFIED` ，比如 ScrollView 和 RecyclerView。在滑动时，实际上是让子 View 在它们的内部滚动，这意味着它们的子 View 的尺寸要大于父 View，所以父 View 不应该对子 View 施加尺寸的约束。
  
  注意，这里的 `SIZE` 是通过 `getChildMeasureSpec` 方法计算出来的，有可能是子 View 在 xml 中设置的尺寸，也有可能是父 View 的尺寸，还有可能是 0。

回到 `getChildMeasureSpec` 方法，该方法的代码如下：

```java
public static int getChildMeasureSpec(int spec, int padding, int childDimension) {
        int specMode = MeasureSpec.getMode(spec);
        int specSize = MeasureSpec.getSize(spec);

        int size = Math.max(0, specSize - padding);

        int resultSize = 0;
        int resultMode = 0;

        switch (specMode) {
        // Parent has imposed an exact size on us
        case MeasureSpec.EXACTLY:
            if (childDimension >= 0) {
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size. So be it.
                resultSize = size;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent has imposed a maximum size on us
        case MeasureSpec.AT_MOST:
            if (childDimension >= 0) {
                // Child wants a specific size... so be it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size, but our size is not fixed.
                // Constrain child to not be bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size. It can't be
                // bigger than us.
                resultSize = size;
                resultMode = MeasureSpec.AT_MOST;
            }
            break;

        // Parent asked to see how big we want to be
        case MeasureSpec.UNSPECIFIED:
            if (childDimension >= 0) {
                // Child wants a specific size... let him have it
                resultSize = childDimension;
                resultMode = MeasureSpec.EXACTLY;
            } else if (childDimension == LayoutParams.MATCH_PARENT) {
                // Child wants to be our size... find out how big it should
                // be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            } else if (childDimension == LayoutParams.WRAP_CONTENT) {
                // Child wants to determine its own size.... find out how
                // big it should be
                resultSize = View.sUseZeroUnspecifiedMeasureSpec ? 0 : size;
                resultMode = MeasureSpec.UNSPECIFIED;
            }
            break;
        }
        //noinspection ResourceType
        return MeasureSpec.makeMeasureSpec(resultSize, resultMode);
    }
```

代码很长，但是实现的功能其实用一句话就可以概括：在不同的父 View 的 `MeasureSpec.MODE`  下，当子 View 的尺寸分别为具体值、`MATCH_PARENT` 和 `WRAP_CONTENT` 的时候，计算出子 View 的 `MeasureSpec` 并返回。所以一共有 3 x 3 = 9 种情况。

这里我不再对每一个情况解释了，代码中的注释已经写的很清楚了。我提一些其他的点：

* **为什么子 View 的 `MeasureSpec` 需要由父 View 共同确定？**

  很大程度上是因为 `MATCH_PARENT` 和 `WRAP_CONTENT` 的存在，因为这两个 dimension 需要借助父 View 才能确定。`MATCH_PARENT` 需要知道父 View 有多大才能匹配到父 View 的大小；而 `WRAP_CONTENT` 虽然表示子 View 的尺寸由自己决定，但是这个大小不能超过父 View 的大小。

  如果所有的 View 的 dimension 只能设置为固定的数值，那么其实子 View 的 `MeasureSpec` 就和父 View 无关了。正如上面代码中，当 `childDimension >= 0` 时，子 View 的 `MeasureSpec` 始终由 `childDimension` 和 `MeasureSpec.EXACTLY` 组成。

* **`AT_MOST` 和 `WRAP_CONTENT` 的关系**

  网上有很多文章说，当一个 View 的尺寸设置为 `WRAP_CONTENT` 时，它的 `MeasureSpec.MODE` 就是 `AT_MOST`。这并不准确。首先，当父 View 的 MODE 是 `UNSPECIFIED` 时，子 View 设置为 `WRAP_CONTENT` 或 `MATCH_PARENT`，那么子 View 的 MODE 也都是 `UNSPECIIED` 而不是 `AT_MOST`。其次，当父 View 是 `AT_MOST` 的时候，子 View 的 `childDimension` 即使是 `MATCH_PARENT`, 子 View 的 MODE 也是`AT_MOST`。所以 `AT_MOST` 与 `WARP_CONTENT` 并不是一一对应的关系。

  看起来有点乱，但是只要始终抓住关键点，即 `AT_MOST` 意味着这个 View 的尺寸有上限，最大不能超过 `MeasureSpec.SIZE` 的值。那具体的值是多少呢？这就要看在 `onMeasure` 中是如何设置 View 的尺寸了。对于一般的视图控件的 `onMeasure` 逻辑，当它的 `MeasureSpec.MODE` 是 `AT_MOST` 的时候，意味着它的大小就是包裹内容的大小，但是最大不能超过 `MeasureSpec.SIZE`，类似于给 View 同时设置 `WRAP_CONENT` 和 `maxHeight`/`maxWidth`。

  **Tips:** 借助`AT_MOST`的特性，可以实现有用的功能。比如需要一个 `WRAP_CONTENT` 的 RecyclerView，它的高度随 item 数目增加而变高，但是有最大高度的限制，超过这个高度不再增加。要实现这样一个 RecyclerView，在 xml 里给 RecyclerView 设置 `android:maxHeight` 是不管用的。但是我们可以继承 RecyclerView 并重写 `onMeasure` 方法，只需要将 

  `heightSpecMode` 改成 `AT_MOST` 即可，如下：

  ```java
  public class MyRecyclerView extends RecyclerView {
      ...
      @Override
      protected void onMeasure(int widthSpec, int heightSpec) {
       		// 构造一个 mode 为 AT_MOST 的 heightSpec，size 为你想要的最大高度，然后传入到 super 中即可
          int newHeightSpec = MeasureSpec.makeMeasureSpec(maxHeight, MeasureSpec.AT_MOST);
          super.onMeasure(widthSpec, newHeightSpec);
      }
  }
  ```

* **`UNSPECIFIED` 什么时候用到？**

  网上很多讲解绘制流程的文章，对于 `UNSPECIFIED` 都是一笔带过，并没有讲得很清楚。`UNSPECIFIED`，顾名思义，不指定尺寸。当一个 View 的 `MeasureSpec.MODE` 是 `UNSPECIFIED` 的时候，说明父 View 对它的尺寸没有任何约束。实际上 android 中使用到 `UNSPECIFIED` 的控件很少，只有 ScrollView、RecyclerView 这类可以滑动的 View 会用到，因为它们的子 View (也就是滑动的内容) 可以无限高，比父 View (视口，ViewPort) 高得多。

  比如 ScrollView 给子 View 施加约束时，就直接构造了一个 `UNSPECIFIED` 的 `MeasureSpec` 来测量子 View：

  ```java
  protected void measureChildWithMargins(View child, int parentWidthMeasureSpec, int widthUsed,
              int parentHeightMeasureSpec, int heightUsed) {
          final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
  
          final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
                  mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                          + widthUsed, lp.width);
          final int usedTotal = mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin +
                  heightUsed;
      // 这里直接构造了一个 UNSPECIFIED 的 MeasureSpec 用来测量子 View
          final int childHeightMeasureSpec = MeasureSpec.makeSafeMeasureSpec(
                  Math.max(0, MeasureSpec.getSize(parentHeightMeasureSpec) - usedTotal),
                  MeasureSpec.UNSPECIFIED);
          child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
      }
  ```

### 三、onMeasure 过程

前面我们明白了 `onMeasure` 方法的入参 `MeasureSpec` 到底是怎样得到的，现在我们把重点转移到 `onMeasure` 过程。

当然你可能会疑惑，之前不是已经得到了 `MeasureSpec` 吗，`MeasureSpec` 内部不是已经有了尺寸的信息吗，为什么还要再测量呢？

首先，`MeasureSpec` 中的尺寸并不能理解成 View 的实际尺寸，`MeasureSpec` 更多的是作为一种父 View 对子 View 测量的**约束**。当子 View 要进行测量时，必须要知道这个约束。而子 View 具体要有多大，是要依赖 `onMeasure` 的逻辑确定的。你当然可以完全不理会这个约束，在 `onMeasure` 中通过 `setMeasuredDimension` 方法随意给 View 设置大小，但是一般是不会这样做的，除非父 View 给你的约束是 `UNSPEECIFIED`。

先来看看最简单的 View.java 中的 `onMeasure` 方法实现：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

可以看到，在 View 的 `onMeasure` 中，是通过 `getDefaultSize` 来确定大小的，然后使用 `setMeasuredDimension` 来设置 View 的宽高。

`getDefaultSize` 方法如下：

```java
public static int getDefaultSize(int size, int measureSpec) {
        int result = size;
        int specMode = MeasureSpec.getMode(measureSpec);
        int specSize = MeasureSpec.getSize(measureSpec);

        switch (specMode) {
        case MeasureSpec.UNSPECIFIED:
            result = size;
            break;
        case MeasureSpec.AT_MOST:
        case MeasureSpec.EXACTLY:
            result = specSize;
            break;
        }
        return result;
    }
```

逻辑很好懂。当 `specMode` 是 `UNSPECIFIED` 的时候，使用 `getSuggestedMinimumWidth`/`getSuggestedMinimumHeight` 返回的尺寸；当 `specMode` 是 `AT_MOST` 或 `EXACTLY` 的时候，使用 `specSize`，即 `MeasureSpec.SIZE`。

从上一节中我们知道，当子 View 的 dimension 是 `WRAP_CONTENT`， 而父 View 的 `specMode` 是 `EXACTLY` 或 `AT_MOST` 的时候，子 View 的 `specMode` 也是 `AT_MOST`。而在上面的代码中，当 View 的 `specMode` 是 `AT_MOST` 的时候，却直接将 `specSize` 返回了，和 `EXACTLY` 处理方式一样。这意味着，View 这个类在 xml 中设置 `WRAP_CONTENT` 和 `MATCH_PARENT` 的效果是一样的。当然这很好理解，因为单纯的 View 类说白了只是一个矩形，并没有“内容”。

不过也正因如此，在我们自己自定义 View 的时候，如果不复写 `onMeasure` 的话，那么 `WARP_CONTENT` 就是没有效果的。所以自定义 View 如果想有 `WRAP_CONTENT` 的效果，那么需要重写 `onMeasure` 并对 `specMode` 为 `AT_MOST` 的情况做处理（当父 View 是可滑动的View 时，`WRAP_CONTENT` 还有可能对应 `UNSPECIFIED` 的 `specMode`, 所以最好也处理这种情况）。

举个例子，比如 TextView 的 `onMeasure` 方法。当我们给 TextView 设置 `WRAP_CONTENT` 的时候，我们很自然的会想让 TextView 的宽度包裹它内部的文字。来看看它的 `onMeasure` 实现：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    ...
    int width;
    ...
    if (widthMode == MeasureSpec.EXACTLY) {
            // Parent has told us how big to be. So be it.
            width = widthSize;
        } else {
        // 如果 widthMode == MeasureSpec.AT_MOST || MeasureSpec.UNSPECIFIED
            if (mLayout != null && mEllipsize == null) {
                des = desired(mLayout);
            }
			...
        	width = des;
            ...
   ...
   setMeasuredDimension(width, height);
}
```

可以看到，当 `widthMode == MeasureSpec.AT_MOST` 时，通过 `desired(Layout layout)` 方法来计算得到 width。`desired` 方法实现如下：

```java
private static int desired(Layout layout) {
        int n = layout.getLineCount();
        CharSequence text = layout.getText();
        float max = 0;

        // if any line was wrapped, we can't use it.
        // but it's ok for the last line not to have a newline

        for (int i = 0; i < n - 1; i++) {
            if (text.charAt(layout.getLineEnd(i) - 1) != '\n') {
                return -1;
            }
        }

        for (int i = 0; i < n; i++) {
            max = Math.max(max, layout.getLineWidth(i));
        }

        return (int) Math.ceil(max);
    }
```

逻辑简单明了，就是取所有文字行的最大行宽。

所以 `onMeasure` 总结起来就是，根据父 View 对自己的约束(`widthMeasureSpec` 和 `heightMeasureSpec`)，结合自身的特性，计算出尺寸并用 `setMeasuredDimension` 赋值。

ViewGroup 的 `onMeasuer` 逻辑和 View 类似，只不过 ViewGroup 一般需要先计算子 View 的尺寸才能确定自身尺寸。比如 LinearLayout `WRAP_CONTENT` 需要先知道子 View 们的高度，加起来才能确定自己多高。

### 四、最顶层的 MeasureSpec

前面我们知道，子 View 在 `onMeasure` 方法中传入的 `measureSpec` 参数是由父 View 的 ` measureSpec` 和 子 View 的 `childDimension` 共同计算得到的约束。而父 View 的 `measureSpec` 是由父 View 的父 View 计算来的。这样形成了一个链条。那这个链条的头在哪里呢，最顶端的根 View 的在 `onMeasure` 的时候它的入参 `measureSpec` 是从哪里来的呢？

对一个 Activity 来说，最顶层的 View 是 `DecorView`，引用关系如下：

> Activity -> WindowManager -> ViewRootImpl -> DecorView

当 Activity 通过 WindowManager 将 `DecorView` 添加到 window 中时，会调用 ViewRootImpl 的 `setView` 方法，持有 `DecorView` ，如下：

```java
// WindowManagerGlobal.java
public void addView(View view, ViewGroup.LayoutParams params,
            Display display, Window parentWindow, int userId) { 
    ...
    root = new ViewRootImpl(view.getContext(), display);

    view.setLayoutParams(wparams);

    mViews.add(view);
    mRoots.add(root);
    mParams.add(wparams);
    ...
}
```

测量过程正是由 ViewRootImpl 发起。也就是说，最顶部的 `DecorView` 在 `measure` 过程中使用的 `measureSpec` 就是由 ViewRootImpl 传入的。这个过程在 ViewRootImple 的 `performTraversals` :

```java
// ViewRootImpl.java
private void performTraversals() {
    if (mWidth != frame.width() || mHeight != frame.height()) {
                mWidth = frame.width();
                mHeight = frame.height();
    }
    ...
    int childWidthMeasureSpec = getRootMeasureSpec(mWidth, lp.width);
    int childHeightMeasureSpec = getRootMeasureSpec(mHeight, lp.height);
    // Ask host how big it wants to be
    performMeasure(childWidthMeasureSpec, childHeightMeasureSpec);
    ...
}

 private static int getRootMeasureSpec(int windowSize, int rootDimension) {
        int measureSpec;
        switch (rootDimension) {

        case ViewGroup.LayoutParams.MATCH_PARENT:
            // Window can't resize. Force root view to be windowSize.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);
            break;
        case ViewGroup.LayoutParams.WRAP_CONTENT:
            // Window can resize. Set max size for root view.
            measureSpec = MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.AT_MOST);
            break;
        default:
            // Window wants to be an exact size. Force root view to be that size.
            measureSpec = MeasureSpec.makeMeasureSpec(rootDimension, MeasureSpec.EXACTLY);
            break;
        }
        return measureSpec;
    }

private void performMeasure(int childWidthMeasureSpec, int childHeightMeasureSpec) {
        if (mView == null) {
            return;
        }
    	mView.measure(childWidthMeasureSpec, childHeightMeasureSpec);
    }
```

可以看到，最顶层的 `measureSpec` 通过 `getRootMeasureSpec` 方法构造的，然后在 `performMeasure` 中传递给 `mView`（就是 `DecorView`）去进行 `measure` 操作。

`getRootMeasureSpec` 方法中，`mWidth/mHeight` 一般是屏幕的宽/高，而 `rootDimension` 是 `DecorView` 的 `layout_width` 和 `layout_height`。这两个值在 WindowManager 调用 `addView` 的时候就已经确定了，是 `MATCH_PARENT`。所以最终得到的是代码 `MeasureSpec.makeMeasureSpec(windowSize, MeasureSpec.EXACTLY);` 构造的 MeasureSpec，也就是说，`DecorView` 的尺寸约束是确切的宽高值，并且是屏幕宽高。这是很合理的，因为最顶层的 View 就应该是屏幕大小。

### 五、总结

* 子 View 在 `onMeasure` 方法中使用的 `MeasureSpec` 来自父 View，而父 View 的 `MeasureSpec` 来自父 View 的父 View，这是一个链条。之所以子 View 尺寸要受到父 View 的约束，是因为 `MATCH_PARENT` 和 `WRAP_CONTENT` 的存在。如果子 View 的尺寸只能设置为固定的 dp 值，那父 View 对子 View 的约束就意义不大了。
* 子 View 的 `MeasureSpec` 是由子 View 的 `dimension` 和父 View 的 `MeasureSpec` 共同计算得来的。子 View 的 `dimension` 就是子 View 在 xml 中设置的 `android:layout_height/android:layout_width`。注意，`WRAP_CONTENT` 和 `MATCH_PARENT`  也是 `dimension`，是尺寸，不是 `MeasureSpec.MODE`，它们是两个概念，不要弄混，虽然这两者之间有关系。
* `WRAP_CONTENT` 和 `AT_MOST` 不是一一对应的关系。`UNSPECIFIED`一般只用于可滑动的 View。
* 虽然 `MeasureSpec` 中已经包含了尺寸约束信息，但是子 View 仍然需要在 `onMeasure` 中进一步确定子 View 具体应该有多大。比如上文提到的 TextView 的例子。一般来说，自定义 View 是需要自己处理 `specMode` 为 `AT_MOST` 的情况的，因为 View 类本身没有处理这个情况，会导致 `WRAP_CONTENT`  失效。
* 在链条最顶端的 `DecorView ` 的 `MeasureSpec` 就是 `ViewRootImpl` 在 `performMeasure` 方法中构造的 `MeasureSpec`。这个 `MeasureSpec` 的 `SIZE`是屏幕宽高，`MODE` 是 `EXACTLY`。