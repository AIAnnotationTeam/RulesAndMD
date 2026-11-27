

# iOS 项目的常用第三方库

## 外部平台能力选型（强制）

先按[统一选型规则](平台能力选择原则.md#外部平台能力选型强制)选择平台，依据 [Cloudflare 能力清单](../平台能力/Cloudflare能力清单.md) 和 [百炼能力清单](../平台能力/百炼平台能力清单.md) 确定适用能力，再选客户端库。系统、本地与界面库保持原职责，云服务 SDK 按项目启用。


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

| 业务能力 | 优先平台路线 | iOS 接入边界 |
| --- | --- | --- |
| 产品认证 | Workers + D1 + Better Auth，按需 Email Sending | 通过已确定的认证协议或适配器接入，验证移动端回调、会话和退出；用户凭据使用 Keychain，不把浏览器 Cookie 流程直接照搬到原生端 |
| 云端业务数据 | Workers API + D1；实时状态按需 DO | 使用 URLSession / 已有网络封装，SwiftData 或 SQLite 继续承担本地离线数据；云端长期凭据不进入 App |
| 文件 | R2 + 后端授权与短期签名 | 上传、下载和播放经受控接口，不把 R2 管理凭据打包到客户端 |
| 服务端业务 | Workers；需系统依赖时使用 Containers | 客户端调用项目 API，不假设兼容 Firebase Callable Functions 的协议 |

| 按选型结果启用的 SDK | 使用边界 |
| --- | --- |
| `Firebase/Core` | 仅实际采用 Firebase 模块时初始化，例如下文 Crashlytics；不因该依赖自动启用 Firebase 全套云服务 |
| `Firebase/Auth`、`Firebase/Firestore` | 仅明确保留现有服务或清单路线无法覆盖所需认证、实时监听与离线同步时启用，记录依据 |
| `Firebase/Storage`、`Firebase/Functions` | 新选型优先 R2 和 Workers；保留旧工程时按业务边界维护，不在新项目无条件安装 |


# 监控与性能工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `Firebase/Crashlytics` | CocoaPods 模板的崩溃诊断基线；保留原生崩溃与非致命错误诊断，不默认叠加 Sentry。Cloudflare Issues 不替代该客户端职责。 |
| `Sentry` | SPM 模板的客户端诊断基线；由 `Observability` Target 统一初始化、脱敏和采样，不默认叠加 Crashlytics。 |
| `Firebase/Performance` | 按客户端性能需求启用，采集启动、网络及业务耗时；服务端请求链路优先使用 Cloudflare Traces，避免重复采集。 |


# 支付与广告分析工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `RevenueCat` | 商店购买、恢复和订阅管理需要时保留；普通 Web 收款按清单国内支付宝、海外 Creem 选型，不将二者等同于商店内购。 |
| `FBSDKCoreKit` | 只为实际需要的 Meta 专属能力引入；仅业务事件采集与转发时优先评估 Analytics Engine / Zaraz HTTP Events API，完整归因不能直接替换。 |


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

| 能力 / 工具 | 接入要求 |
| --- | --- |
| 百炼实时对话 / 同传 | 优先按百炼清单分别确定对话与翻译模型，由服务端持有长期密钥；iOS 使用项目语音 Adapter、受控网关或经验证的短期凭证接入 |
| URLSession WebSocket / Starscream | 承担协议传输，采集与播放使用平台音频能力；Adapter 处理供应商事件、音频格式、打断、重连和错误映射，验证后确定实现 |
| `swift-realtime-openai`（产品名 `RealtimeAPI`） | 仅已确认采用 OpenAI Realtime 或维护旧接入时使用，不作为新项目默认依赖；不能只改地址就视为百炼兼容 |







