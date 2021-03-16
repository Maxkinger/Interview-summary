## 一文读懂 onMeasure

记得我刚接触自定义 View 的时候，关于 View 的测量、布局、绘制三大流程，最难懂的就是 `onMeasure` 过程。相比于 `onLayout` 和 `onDraw` 只关注当前 View 的逻辑，`onMeasure` 往往要处理父 View 和子 View 之间关系，这让 `onMeasure` 理解和实践起来都变得更难了一些。当时并没有理解的很透彻，不过还是能做的。后来随着经验提高，慢慢搞清楚了 `onMeasure` 过程，并且发现网上很多关于 `onMeasure` 的一些说法都是错的，或者说不准确的。

### 一、MeasureSpec 来自哪里

我们看看 View 的 `onMeasure` 方法：

```java
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
                getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
    }
```

我们都知道在自定义 View 的时候要重写 `onMeasure` 方法，要对方法的两个参数 `widthMeasureSpec` 和 `heightMeasureSpec` 进行处理。网上很多文章在提到 `measureSpec` 的时候会详细的说它的概念，比如 32 位整数，头两位是 mode，后面 30 位是 size。但是往往都是罗列一大串知识点，并没有理清其中的关系和逻辑。所以这篇文章我想换一种思路来讲 `onMeasure` ，尽量避免罗列知识点，而是从我自己的理解出发，写清楚 `onMeasure` 的底层逻辑，以及为什么这样做。

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

`widthMeasureSpec` 和 `heightMeasureSpec` 是作为父 View 对子 View 施加的约束传入到 `measure` 方法中的。当然这个约束并不完全由父 View 产生，而是由父 View 和子 View 的尺寸共同作用得到的。这个后面会讲到。

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

最后一步的 `measureChildWithMargins` 内部就调用了子 View 的 `measure` 方法，如下，这个方法其实是 ViewGroup 这个基类中的方法：

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

**注意**：网上的很多文章，在看的时候很容易让人模糊了 `MATCH_PARENT` 及 `WRAP_CONTENT`  这两者与 `MeasureSpec.MODE` 的关系。一些文章会说，当 View 的尺寸设置为 `WRAP_CONTENT` 的时候，它的 mode 对应 `AT_MOST`。这种说法并不准确。 `MATCH_PARENT` 和`WRAP_CONTENT` 就是 size，它和 `measureSpec` 的 mode 是两个不同的概念，只不过它是标记用的 size，需要借助父 View 给出具体的尺寸。

在深入 `getChildMeasureSpec` 的具体实现之前，我们回顾一下 `MeasureSpec` 的概念。

