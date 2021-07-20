 <center>
     <h3>郭哲轩</h3>
     <div>
         <span>
             13002057726
         </span>
         |
         <span>
         	 jiujiuli@qq.com
         </span>
 </center>


** **

#### <img src="assets/info-circle-solid.svg" width="25px"> 个人信息

​	基本信息：男 / 1996 	 									  求职意向：Android 研发工程师

​	工作经验：2 年													 期望薪资：20k-25k

​	Blog：https://ultimatehandsomeboy666.github.io/ 或者 https://juejin.cn/user/413072104102776

​	Github：https://github.com/ultimateHandsomeBoy666

#### <img src="assets/briefcase-solid.svg" width="25px"> 工作经历

​	**深圳柔宇科技有限公司**																					   **2020.10 - 至今**

​	**柔派折叠屏手机负一屏 Android 客户端**

* 产品简介：负一屏展示多种信息卡片，如行程、日程、股票、收藏相册等，顶部提供全局搜索，底部展示新闻 feed 流。主要负责短信行程等业务开发、交互/性能优化以及各类 bug 修复工作。
* 技术架构：应用首页为 Service 而非 Activity，通过 WindowManager 展示 View，通过 IPC 方式与 Launcher 交互。整体技术架构偏老旧，MVC + java，部分代码耦合度较高，有解耦需求。在保证稳定性的情况下，对新增业务需求引入了 kotlin 和 mvvm 组件，实现数据绑定及代码解耦。
* 优化/难点
  * NestedScrollingParent/NestedScrollingChild 实现顶部搜索、卡片列表、新闻列表的多级嵌套滑动，支持 fling。减少新闻列表打开/关闭切换的卡顿，约 1~2 秒。
  * 减少数据的重复拉取，合理增加相册缓存，降低内存占用和抖动。
  * 基于字节码压缩、图片压缩、资源混淆的 APK 瘦身，减少 16% 的体积。
  * 基于有限状态机的埋点，匹配复杂埋点需求。

​	**深圳芒果未来教育科技有限公司**											                            **2019.6 - 2020.7**

​	**一起练琴 Android 客户端（国内 + 海外）**

- 产品简介：音乐陪练细分领域的第一梯队产品。核心功能为采集练琴者发出的声音，通过自研算法采集声音合成 PCM ，进行音频分析，光标实时跟随乐谱，音频上传算法服务器，远端返回练习结果。另外还包括乐谱标注、在线视频陪练、屏幕共享、打卡等功能。本人负责相关业务开发，以及线上问题修复，还有部分性能优化。
- 技术架构：MVVM 架构，kotlin/java + Viewmodel + Livedata + Rxjava，通过 JNI 与音频算法交互，实现乐谱播放、乐音采集等功能。使用 vexflow 绘制乐谱，通过 webview jsBridge 与之交互。
- 优化/难点
  * 降低乐谱绘制白屏率 50%，减少线上 crash 约 60%。
  * 自定义 WaveView 等展示打卡进度，支持液面重力倾斜。
  * 配置正确符号表，定位 native crash 修复。
  * 实现轻量级线上卡顿监控，日志上报等功能。
  * 采用 AppBundle、图片压缩、代码混淆，动态下发 so 库等方式瘦身 Apk 约 40M。
  * 使用堆外内存存放乐谱，优化 Java 内存占用。
  * 基于私有协议实现的视频乐器陪练课堂场景，音视频服务由第三方提供。
  * 基于 Arouter 组件化的应用架构调整。

#### <img src="assets/tools-solid.svg" width="25px"> 技能清单

* 熟悉 Java、kotlin 语言， Android sdk，常用 Jetpack 组件。熟悉多线程，了解协程。熟悉自定义 View。熟悉常用性能优化手段。
* 了解 Flutter 以及 Jetpack compose。
* 英语水平好，可以无障碍阅读文档，常去 medium、stackoverflow 取经。

#### <img src="assets/graduation-cap-solid.svg" width="25px"> 教育经历

​	大连理工大学 - 化学工程  本科																		 2013.9 - 2017.7

