# 2018 Google I/O 大会汇总

2018年Google I/O 开发者大会在山景城的Shoreline Amphitheatre（圆形剧场）如期在北京时间5月9日凌晨1点举办，我没有直接的观看直播,在今天一早就看最新的消息并整理；

## Android P

Android P 引入了 ML Kit，这是一个新的软件开发工具包（SDK），允许开发者将大量 Google 预先建立的机器学习模型整合到他们的 Android 或 iOS 应用中。模型包括支持文本识别、人脸检测、条码扫描、图像标记和地标识别等等，并且可以在线和离线使用。
## Android Jetpack

Google 还发布了 Android Jetpack，并称其为下一代的 Android 组件，旨在帮助开发者加快应用开发速度。Android Jetpack 将支持库向后兼容和立即更新的优点融合到更多组件中，让开发者能够快速轻松地开发出拥有卓越性能的高质量应用。它能够处理类似后台任务、UI 导航以及生命周期管理之类的活动，免去开发者编写样板代码的麻烦，专注提升应用体验。

Android Jetpack 组件包括以下 4 个部分：WorkManager、Paging、Navigation 以及 Slices，能完美兼容 Kotlin 语言，利用 Android KTX 大幅节省代码量。
## Kotlin

Google 表示，自去年宣布支持 Kotlin 以来，该语言受到开发者社区的广泛认可。95% 的开发者表示很喜欢用 Kotlin 进行 Android 的开发，Play Store 中用 Kotlin 开发的应用在去年增至 6 倍，在高级开发者中有 35% 的人选择使用 Kotlin 进行开发，而且这个数字正在逐月递增。

Google 会继续改善 Kotlin 在支持库、工具、运行时 (runtime)、文档以及培训中的开发体验。Google 在今年2月发布的 Android KTX，也会包含在上面提到的 Android Jetpack 中，力图优化 Kotlin 开发者体验；同时继续改善 Android Studio、Lint 支持以及 R8 优化中的工具；而且对 Android P 中的运行时 (Android Runtime) 进行微调，以此加快 Kotlin 编写的应用的运行时间。

## Android Studio 3.2 金丝雀版

Android Studio 3.2 引入了 Android Jetpack 支持工具，包括一款视觉导航编辑器以及全新代码重构工具。金丝雀版本同时还包含了可用于创建全新的 Android App Bundle 格式的构建工具、用于快速启动 Android 模拟器的快照功能 (Snapshot)、给下载及安装包瘦身的新 R8 优化器、以及用于测量应用对电池续航影响的新电量分析工具 (Energy Profiler) 等等。

最新版本的 Android Studio 3.2 [可点此下载](https://developer.android.com/studio/preview/)。

## Android App Bundle 以及 Google Play Dynamic Delivery (动态交付)

Google 向 Android 引入了新 App 模式。利用全新发布格式 —— Android App Bundle，大幅度减少应用体积。现在只须在 Android Studio 中构建一个应用束 (app bundle)，就可以将应用所需的全部内容 (适用于所有设备) 都涵盖在内：所有语言、所有设备屏幕大小、所有硬件架构。

## Android Things 1.0

Android Things 作为 Google 旗下的一款操作系统 (OS)，能够帮助开发者规模化开发和维护物联网设备。Google 表示此前推出的开发者预览版的 SDK 下载次数已经突破 10 万，Android Things 1.0 将在本周与各位开发者见面。

Android Things 平台添加了对 3 种新系统模组 (System-on-Modules 或 SoMs) 的支持，并承诺在接下来的三年中提供长期支持，并让开发者自行决定是否需要扩展支持，帮助他们更容易地设计出原型并推向市场。同时还推出了一个 Android Things 控制台 (Android Things Console) ，帮助开发者定期获取 Google 最新稳定性修复包以及安全升级包，从而实现从发布、管理到设备更新的无缝连接。

现在是AI和物联网时代，自己大学也是学习的嵌入式所以对Android Things 还是有很大的兴趣的。

[ Android Things 官网地址](https://developer.android.google.cn/things/)