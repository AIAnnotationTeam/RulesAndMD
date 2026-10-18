

 iOS 项目的常用第三方库


# 网络通信工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `Alamofire` | Swift HTTP 网络库，提供接口请求、文件上传下载、请求取消和流式响应等能力。 |
| `Moya` | 基于 Alamofire 的网络抽象层，通过枚举和协议统一定义接口地址、参数、请求方法和响应处理。 |
| `Starscream` | Swift WebSocket 客户端，用于客户端与服务端之间的实时双向通信。 |


# 数据解析工具

使用原生 Codable


# 本地存储工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `SQLite.swift` | Swift SQLite 数据库封装，提供类型安全的建表、查询、插入、更新和删除操作。 |
| `MMKV` | 腾讯开源的高性能键值存储库，用于保存用户配置、引导流程标记和本地状态。 |


# 界面布局与图片工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `SnapKit` | Swift Auto Layout 布局工具，通过简洁的链式语法编写和更新界面约束。 |
| `Kingfisher` | 图片下载与缓存工具，支持异步加载、内存和磁盘缓存、占位图及图片处理。 |
| `SkeletonView` | 骨架屏组件，在数据加载期间展示占位视图，支持渐变和加载动画。 |


# 动画与交互组件

| 第三方库 | 简要介绍 |
| --- | --- |
| `lottie-ios` | 播放 Lottie JSON 动画，用于启动引导、加载状态、图标和庆祝效果等界面。 |
| `Toast-Swift` | 轻量提示组件，为 UIView 提供 Toast 消息和加载提示。 |
| `SwiftMessages` | 自定义消息提示组件，支持顶部横幅、底部通知、弹出卡片和交互式提示。 |
| `FDFullscreenPopGesture` | 为导航控制器提供全屏侧滑返回手势。 |
| `IQKeyboardManagerSwift` | 自动管理键盘与输入框的位置关系，减少键盘遮挡，并提供输入辅助工具栏。 |
| `JXSegmentedView` | 分段菜单组件，支持标题切换、指示器动画及与分页内容联动。 |
| `PullToRefreshKit` | 下拉刷新和上拉加载组件，支持自定义刷新视图及动画。 |
| `MJRefresh` | 列表刷新组件，提供下拉刷新、上拉加载和自定义刷新状态展示。 |


# 富文本与 Markdown 工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `DTCoreText` | 基于 Core Text 的富文本处理工具，支持将 HTML 转换为富文本并进行排版展示。 |
| `Down` | 基于 cmark 的 Markdown 解析与渲染工具，支持转换为 HTML 和 NSAttributedString。 |


# 音频与蓝牙工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `SCSiriWaveformView` | Siri 风格动态音频波形组件，用于展示录音或语音交互状态。 |
| `FDWaveformView` | 音频文件波形组件，支持波形绘制、播放进度高亮及缩放等交互。 |
| `BlueToothTool` | 项目通过 Git 仓库引入的蓝牙与音频工具库，用于相关设备连接和音频能力封装；具体接口以该仓库实现为准。 |


# 身份认证与云端数据工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `Firebase/Core` | Firebase 基础集成入口，用于初始化 Firebase 应用及相关基础能力。 |
| `Firebase/Auth` | 用户认证 SDK，提供登录、账号管理、身份令牌及认证状态管理。 |
| `Firebase/Firestore` | 云端文档数据库 SDK，提供业务数据读写、查询、实时监听和事务操作。 |
| `Firebase/Storage` | 云端对象存储 SDK，用于上传、下载和管理录音、图片等文件。 |
| `Firebase/Functions` | Cloud Functions 客户端 SDK，用于调用云端 Callable Functions，执行服务端业务逻辑。 |


# 监控与性能工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `Firebase/Crashlytics` | 崩溃监控工具，收集崩溃、非致命错误及诊断信息，帮助定位线上问题。 |
| `Firebase/Performance` | 性能监控工具，采集应用启动、网络请求及自定义业务流程的耗时指标。 |


# 支付与广告分析工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `RevenueCat` | 应用内购买与订阅管理 SDK，提供商品查询、购买、恢复购买和用户权益管理。 |
| `FBSDKCoreKit` | Meta/Facebook 基础 SDK，提供应用事件上报、广告效果衡量及相关平台集成能力。 |


# Swift 扩展工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `SwifterSwift/SwiftStdlib` | Swift 标准库扩展，为字符串、数组、字典等类型提供便捷方法。 |
| `SwifterSwift/Foundation` | Foundation 扩展，为日期、URL、Data 等类型提供常用工具方法。 |
| `SwifterSwift/CoreGraphics` | Core Graphics 扩展，简化 CGPoint、CGSize、CGRect 等几何类型的操作。 |
| `SwifterSwift/Dispatch` | Dispatch 扩展，提供线程调度和任务执行相关的便捷方法。 |


# 开发调试工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `LookinServer` | 配合 Lookin 桌面客户端检查运行中的界面层级、视图属性和布局；当前仅在 Debug 配置中启用 `SwiftAndNoHook` 子模块。 |


# AI 实时语音工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `swift-realtime-openai`（产品名 `RealtimeAPI`） | OpenAI Realtime API 的 Swift 客户端封装，提供实时会话、音频交互和事件处理能力。项目通过 `Conversation` 等接口实现 AI 角色扮演语音对话。 |







