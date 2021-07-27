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
       	 |
         <span>
         	 Android 开发工程师
         </span>
 </center>

** **

#### <img src="assets/info-circle-solid.svg" width="25px"> 个人信息

​	基本信息：男 / 1996 

​	工作年限：2 年

​	Blog：https://ultimatehandsomeboy666.github.io/ 或者 https://juejin.cn/user/413072104102776

​	Github：https://github.com/ultimateHandsomeBoy666

#### <img src="assets/briefcase-solid.svg" width="25px"> 工作经历

##### 深圳柔宇科技有限公司																					   **2020.10 - 至今**

​	**负一屏 Android 客户端**

* 产品简介：负一屏展示多种信息卡片，如行程、日程、股票、收藏相册等，顶部提供全局搜索，底部展示新闻 feed 流。主要负责短信行程等业务开发、交互/性能优化以及各类 bug 修复工作。
* 技术架构：应用主页为 Service 而非 Activity，通过 WindowManager 展示 View，IPC 与 Launcher 交互。整体技术架构偏老旧，MVC + java，部分代码耦合度较高，有解耦需求。在保证稳定性的情况下，对新增业务需求引入了 kotlin 和 mvvm 组件，实现数据绑定及代码解耦。
* 优化/难点
  * NestedScrollingParent/NestedScrollingChild 实现顶部搜索、卡片列表、新闻列表的多级嵌套滑动，支持 fling。
  * 定位新闻流显示/隐藏切换的卡顿问题，并优化，减少 1~2 秒卡顿。
  * 减少数据的重复拉取，合理增加相册缓存，降低内存占用和抖动。
  * 基于字节码压缩、图片压缩、资源混淆的 APK 瘦身，减少 16% 的体积。
  * 基于有限状态机的埋点，匹配复杂埋点需求。

​    **全局搜索 Android 客户端**

* 产品简介：全局搜索可以从桌面通过手势拉起，在用户输入关键字之后，搜索应用包、文件、设置项、短信、联系人、通话、备忘录等相关内容并列表展示。
* 技术架构：主页运行在 Service 中，通过 WindowManager 加载 View。应用从零开始搭建，采用了 kotlin + Databinding + Livedata + Rxjava + ViewModel 的 mvvm 架构。使用 ContentResolver + 匹配规则进行全局的搜索并得到结果。
* 优化/难点
  * 一个月的短周期内业务的快速开发，迭代，bug 修复，持续集成，交付。
  * **IO查询速度如何优化的？缓存？索引？paging？**

##### 深圳芒果未来教育科技有限公司											                            **2019.6 - 2020.7**

​	**一起练琴 Android 客户端（国内 + 海外）**

- 产品简介：音乐陪练细分领域的第一梯队产品。核心功能为采集练琴者发出的声音，通过自研算法采集声音合成 PCM ，进行音频分析，光标实时跟随乐谱，音频上传算法服务器，远端返回练习结果。另外还包括乐谱标注、在线视频陪练、屏幕共享、打卡等功能。本人负责相关业务开发，以及线上问题修复，以及性能优化工作。
- 技术架构：MVVM 架构，kotlin/Java + Viewmodel/Repository + Livedata + Rxjava，通过 JNI 与音频算法交互，实现乐谱播放、乐音采集等功能。使用 vexflow 绘制乐谱，通过 webview jsBridge 与之交互。
- 优化/难点
  * **如何定位问题的**？？？降低乐谱绘制白屏率 50%，减少线上 crash 约 60%。
  * 自定义 ProgressBar、 WaveView 等展示打卡进度，WaveView 支持液面重力倾斜。
  * 配置正确符号表，定位 native crash 修复。
  * 实现轻量级线上卡顿监控，日志上报等功能。
  * 采用 AppBundle、图片压缩、代码混淆，动态下发 so 库等方式瘦身 Apk 约 40M。
  * 使用堆外内存存放乐谱，优化 Java 内存占用。
  * 基于私有协议实现的视频乐器陪练课堂场景，音视频服务由第三方提供。
  * 基于 Arouter 组件化的应用架构调整。

#### <img src="assets/Github.svg" width="30px"> 开源项目

#####   ultimateHandsomeBoy666/Particle

	* 一个使用 kotlin 编写的酷炫的粒子动画库，支持多种形状、位图、矢量图。支持自定义粒子轨迹。

#####   ultimateHandsomeBoy666/Glance (fork from guolindev/Glance)

* 一个在手机上调试数据库的工具，作为 contributer 参与。

#### <img src="assets/tools-solid.svg" width="25px"> 技能清单

* 熟悉 Java、kotlin 语言， Android sdk，常用 Jetpack 组件。了解 Flutter 以及 Jetpack compose。
* 熟悉多线程，了解协程。熟悉自定义 View。熟悉常用性能优化手段。
* 熟悉常用数据结构和算法，熟悉常用操作系统、计算机网络知识。
* 英语水平好，可以无障碍阅读文档，常去 medium、stackoverflow 取经。

#### <img src="assets/graduation-cap-solid.svg" width="25px"> 教育经历

​	大连理工大学 - 化学工程  本科																		 2013.9 - 2017.7

（亲爱的面试官和 hr，为了避免误会，我并不是为了蹭 985 而学的化工。实际上作为大工的王牌专业，化工在我入学的时候分数线比计算机类高。）

