## 一个酷炫的 android 粒子动画库

### 一、灵感

做这个粒子动画库的灵感来自于 MIUI 卸载应用时的动画：

<img src="%E4%B8%80%E4%B8%AA%E9%85%B7%E7%82%AB%E7%9A%84%20android%20%E7%B2%92%E5%AD%90%E5%8A%A8%E7%94%BB%E5%BA%93.assets/ezgif-7-ce4e3b4c2758.gif" alt="ezgif-7-ce4e3b4c2758" style="zoom:30%;" />

这个爆炸的粒子效果看起来很酷炫，而且粒子颜色是从 icon 中拿到的。

最开始我简单实现了类似爆炸的效果，后来想到可以直接扩展一下，写一个通用的粒子动画库。

### 二、使用

项目地址：https://github.com/ultimateHandsomeBoy666/Particle

Particle 是一个使用 kotlin 编写的粒子动画库，可以用几行代码轻松搞定一个粒子动画。同时也支持高度自定义的粒子动画轨迹，可以打造出非常炫酷的自定义动画。这个项目发布了 0.1 版本在 JitPack 上，按如下操作引入：

在根目录的 `build.gradle` 中的 `allprojects` 中添加(注意不是 `buildScript`)：

```groovy
allprojects {
	repositories {
		...
		maven { url 'https://jitpack.io' }
	}
}
```

然后在你的项目中引入依赖即可。

```gr
implementation 'com.github.ultimateHandsomeBoy666:Particle:0.1'
```

在引入了 Particle 之后，只需要下面几行简单的代码，就可以实现上面的粒子爆炸效果：

```kotlin
Particles.with(context, container) // container 是粒子动画的宿主父 ViewGroup
		.colorFromView(button)// 从 button 中采样颜色
		.particleNum(200)// 一共 200 个粒子
		.anchor(button)// 把 button 作为动画的锚点
		.shape(Shape.CIRCLE)// 粒子形状是圆形
		.radius(2, 6)// 粒子随机半径 2~6
		.anim(ParticleAnimation.EXPLOSION)// 使用爆炸动画
		.start()
```

<img src="%E4%B8%80%E4%B8%AA%E9%85%B7%E7%82%AB%E7%9A%84%20android%20%E7%B2%92%E5%AD%90%E5%8A%A8%E7%94%BB%E5%BA%93.assets/ezgif-7-f134d41024b9.gif" alt="ezgif-7-f134d41024b9" style="zoom:33%;" />

### 三、粒子形状

粒子的形状支持**圆形、三角形、矩形、五角星以及矢量图形及位图，并且支持多种图形粒子混合。

下面详细说明。

#### `Shape.CIRCLE` 和 `Shape.HOLLOWCIRCLE`

* 圆形和空心圆

* 使用 `radius`  定义圆的大小。空心圆使用 `strokeWidth` 定义粗细。

#### `Shape.TRIANGLE` 和 `Shape.HOLLOWTRIANGLE`

* 实心三角形和空心三角形

* 使用 `width` 和 `height`  定义三角形的大小。空心三角形使用 `strokeWidth` 定义粗细。

#### `Shape.RECTANGLE` 和 `Shape.HOLLOWRECTANGLE`

* 实心矩形和空心矩形。

* 使用 `width` 和 `height`  定义矩形的大小。空心矩形使用 `strokeWidth` 定义粗细。

#### `Shape.PENTACLE` 和 `Shape.HOLLOWPENTACLE`

* 实心五角星和空心五角星

* 使用 `radius`  定义五角星外接圆的大小。空心五角星使用 `strokeWidth` 定义粗细。

#### `Shape.BITMAP`

* 支持位图。

* 支持矢量图，只需要把矢量图 xml 的资源 id 传入即可。
* 图片粒子不受 color 设置的影响。

除了上述单种图形以外，还支持任意图形的混合粒子：

<img src="%E4%B8%80%E4%B8%AA%E9%85%B7%E7%82%AB%E7%9A%84%20android%20%E7%B2%92%E5%AD%90%E5%8A%A8%E7%94%BB%E5%BA%93.assets/ezgif-7-c8c75ee72e6c.gif" alt="ezgif-7-c8c75ee72e6c" style="zoom:33%;" />

### 四、粒子动画

#### 动画控制

粒子的动画使用 `ValueAnimator` 来控制，可以自行定义 animator 来控制动画的行为，包括动画时长、Interpolater、重复、开始结束的监听等等。

#### 粒子特效

目前仅支持粒子在运动过程中的旋转，如下。后续会增加更多效果

<img src="%E4%B8%80%E4%B8%AA%E9%85%B7%E7%82%AB%E7%9A%84%20android%20%E7%B2%92%E5%AD%90%E5%8A%A8%E7%94%BB%E5%BA%93.assets/ezgif-7-4c9eb5782451.gif" alt="ezgif-7-4c9eb5782451" style="zoom:33%;" />

#### 粒子轨迹

粒子轨迹的控制使用 `IPathGenerator` 接口的派生类来完成。库中自带四种轨迹动画，分别是：

* `ParticleAnimation.EXPLOSION`  爆炸💥效果
* `ParticleAnimation.RISE` 粒子上升
* `ParticleAnimation.FALL` 粒子下降
* `ParticleAnimation.FIREWORK` 烟花🎇效果

如果想要自定义粒子运动轨迹的话，可以继承  `IPathGenerator`  接口，复写生成粒子坐标的方法：

```kotlin
private fun createPathGenerator(): IPathGenerator {
  // LinearPathGenerator 库中自带
        return object : LinearPathGenerator() {
            val cos = Random.nextDouble(-1.0, 1.0)
            val sin = Random.nextDouble(-1.0, 1.0)

            override fun getCurrentCoord(progress: Float, duration: Long): Pair<Int, Int> {
              // 在这里写你想要的粒子轨迹
                val originalX = distance * progress
                val originalY = 100 * sin(originalX / 50)
                val x = originalX * cos - originalY * sin
                val y = originalX * sin + originalY * cos
                return Pair((0.01 * x * originalY).toInt(), (0.008 * y * originalX).toInt())
            }
        }
    }
```

然后把这个返回 `IPathGenerator` 的方法通过高阶函数的形式传入即可：

```kotlin
particleManager!!.colorFromView(button)
                .particleNum(300)
                .anchor(it)
                .shape(Shape.CIRCLE, Shape.BITMAP)
                .radius(8, 12)
                .strokeWidth(10f)
                .size(20, 20)
                .rotation(Rotation(600))
                .bitmap(R.drawable.ic_thumbs_up)
                .anim(ParticleAnimation.with({
                  // 控制动画的animator
                    createAnimator()
                }, {
                  // 粒子运动的轨迹
                    createPathGenerator()
                })).start()
```

上述代码中的 `ParticleAnimation.with` 方法接受两个高阶函数分别生成动画控制和粒子轨迹。

```kotlin
fun with(animator: () -> ValueAnimator = DEFAULT_ANIMATOR_LAMBDA,
        generator: () -> IPathGenerator): ParticleAnimation {
    return ParticleAnimation(generator, animator)
}
```

终于，经过上面的折腾，可以得到下面的酷炫动画：

<img src="%E4%B8%80%E4%B8%AA%E9%85%B7%E7%82%AB%E7%9A%84%20android%20%E7%B2%92%E5%AD%90%E5%8A%A8%E7%94%BB%E5%BA%93.assets/ezgif-7-07fabd0e2118.gif" alt="ezgif-7-07fabd0e2118" style="zoom:33%;" />

当然，只要你想要，可以构造出无限多的粒子动画轨迹，不过这可能要求一点数学功底🐶。

在 https://github.com/ultimateHandsomeBoy666/Particle 目录下有一份我之前试验的比较酷炫的轨迹公式合集，可以参考。

### 五、注意事项

* 粒子动画比较消耗内存和 CPU，所以粒子数目太多，比如超过 1000 的话，可能会有卡顿。
* 默认在动画结束的时候，粒子是不会消失的。如果要让粒子在动画结束时消失，可以自定义 `ValueAnimator` 监听动画结束，在结束时调用 `ParticleManager.hide()` 方法来隐藏粒子。
* 如果需要反复触发粒子动画，比如按一次按钮触发一次，可以使用一个全局的 `particleManager` 变量来启动和取消粒子动画，可以避免内存消耗和内存抖动。比如：

```kotlin
particleManager = Particles.with(this, container)
button.setOnClickListener {
        particleManager!!
  				.colorFromView(button)
          .particleNum(300)
          .anchor(it)
          .shape(Shape.CIRCLE, Shape.BITMAP)
          .radius(8, 12)
          .rotation(Rotation(600))
          .anim(ParticleAnimation.EXPLOSION)

        particleManager!!.start()
}
```

### 六、最后

这个项目暂时除了第一个版本，也是我自己第一个开源发布的项目，会花一些时间精力去维护。

* 后续会增加更多粒子特效，比如粒子运动过程中的尺寸、透明度、颜色等改变
* 再后续可能会出一份 compose 版本🐶

最后，https://github.com/ultimateHandsomeBoy666/Particle 项目地址在这里，欢迎 fork、pr、issues、star，求求大家点个小星星吧✨❤️，感谢！

