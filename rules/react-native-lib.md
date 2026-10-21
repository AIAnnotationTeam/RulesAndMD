# React Native 项目的常用第三方库

本清单面向 TypeScript 裸 React Native 中大型项目，覆盖 iOS、Android，与 [React Native 架构规范](react-native-架构.md) 保持一致。固定采用 React Navigation、TanStack Query、Zustand、React Hook Form、Zod、MMKV 和 Keychain/Keystore，不引入承担相同职责的平行框架。

基础依赖构成工程基线；业务能力库仅在对应功能存在时安装。第三方 SDK 经 Core 适配或 Feature Data 层接入，领域层保持纯 TypeScript。

## 选型与调研口径

| 项目 | 固定口径 |
| --- | --- |
| 核查日期 | 2026-10-21 |
| 工程范围 | 裸 React Native、TypeScript、iOS、Android、新架构；不把 Expo Go 作为运行环境 |
| 使用量 | 使用 npm 实际返回的周下载量判断使用规模；包括 CI、重复安装和间接依赖，不等于用户数或生产项目数 |
| 指标范围 | TanStack Query、Zustand、Axios 等为跨生态包，其下载量包含 Web/Node.js 使用，不能视为 React Native 专属份额 |
| 社区活跃度 | 结合仓库维护记录、维护主体、官方文档和 npm 弃用标记；最近推送可能包含机器人更新，不能单独代表发布质量 |
| 信息限制 | 部分 GitHub 请求触发限流，仅列实际成功取得的记录；未查询到的数据不填估算值 |
| 基线与扩展 | 导航、状态、网络、表单和存储统一选型；相机、音视频、通知和付费等按功能安装 |
| 版本锁定 | 根据项目 React Native、React、Node、原生工具链和 peerDependencies 选择兼容稳定版本，提交包管理器锁文件 |
| 原生集成 | 原生依赖变更后重新构建二进制；iOS 完成 Pods 集成，Android 完成 Gradle 与 Manifest 配置 |

## 导航与安全区域工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [@react-navigation/native](https://www.npmjs.com/package/@react-navigation/native) | React Navigation 核心，提供导航容器、路由状态和深链集成。 | 基础依赖；导航集中于 `src/app/navigation`，Screen 实现仍属于 Feature。 |
| [@react-navigation/native-stack](https://www.npmjs.com/package/@react-navigation/native-stack) | 基于原生页面容器的栈导航，提供原生转场及返回行为。 | 基础导航器；承担详情、编辑和认证等栈式页面。 |
| [@react-navigation/bottom-tabs](https://www.npmjs.com/package/@react-navigation/bottom-tabs) | 底部标签导航，组织主要业务区域。 | 项目存在底部标签时引入；不在导航器内实现业务页面。 |
| [react-native-screens](https://www.npmjs.com/package/react-native-screens) | 导航所用的原生 Screen 容器基础设施。 | 与 React Navigation 和 React Native 使用兼容版本，不单独追最新。 |
| [react-native-safe-area-context](https://www.npmjs.com/package/react-native-safe-area-context) | 提供安全区域 Insets，适配刘海、状态栏与系统手势区域。 | 基础依赖；根部装配 Provider，避免多层重复叠加安全区间距。 |

## 服务端状态与客户端状态工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [@tanstack/react-query](https://www.npmjs.com/package/@tanstack/react-query) | 服务端状态工具，管理异步请求缓存、失效、重试、分页与 Mutation。 | 基础依赖；Query Key 与请求函数按 Feature 归属；不把响应整体复制进 Zustand。 |
| [zustand](https://www.npmjs.com/package/zustand) | 轻量客户端状态管理，支持选择性订阅和中间件。 | 基础依赖；Feature Store 放在 `application/state`；页面局部状态使用 React `useState`。 |

TanStack Query 的 `onlineManager` 接入 NetInfo，`focusManager` 接入 React Native `AppState`。原生 App 的前后台及网络状态不能直接沿用浏览器事件模型。持久化服务端缓存属于独立工程能力，不与普通 MMKV 配置存储混为一谈。

## 网络通信工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [axios](https://www.npmjs.com/package/axios) | HTTP 客户端，支持拦截器、超时、请求取消和响应转换。 | 基础依赖；客户端位于 `core/api`，Feature Data 层定义 API；Token 刷新集中协调，禁止无限重试。 |
| [@react-native-community/netinfo](https://www.npmjs.com/package/@react-native-community/netinfo) | 提供设备网络连接状态和可达性相关信息。 | 网络状态与 Query 重连适配；连接状态不能代替目标业务服务的实际请求结果。 |

普通 WebSocket 使用 React Native 提供的 `WebSocket` API；只有服务端使用特定协议时才集成相应协议客户端，不把所有实时场景都当成 Socket.IO。

## 表单与数据校验工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [react-hook-form](https://www.npmjs.com/package/react-hook-form) | 管理表单值、校验状态、提交和字段订阅。 | 基础依赖；通过 `Controller` 或 `useController` 连接原生输入控件，表单状态不进入 Zustand。 |
| [zod](https://www.npmjs.com/package/zod) | TypeScript 优先的运行时 Schema 校验和类型推导。 | 基础依赖；校验表单、环境变量及外部输入，API 校验 Schema 留在 Data 层；Domain 不依赖该库。 |
| [@hookform/resolvers](https://www.npmjs.com/package/@hookform/resolvers) | 把 Zod 校验结果接入 React Hook Form。 | 与表单库、Zod 配套，统一使用 `zodResolver`；不同时安装另一套表单校验框架。 |

## 本地存储与敏感信息工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [react-native-mmkv](https://www.npmjs.com/package/react-native-mmkv) | MMKV 原生键值存储封装，用于偏好、轻量缓存和客户端状态持久化。 | 基础依赖；通过 `core/storage` 适配；同步访问不能变成大对象高频序列化。 |
| [react-native-nitro-modules](https://www.npmjs.com/package/react-native-nitro-modules) | Nitro Modules 原生模块运行基础设施，供 MMKV 当前主版本等库使用。 | MMKV 配套依赖；不是另一个存储库，也不等同于 Reanimated Worklets。 |
| [react-native-keychain](https://www.npmjs.com/package/react-native-keychain) | 访问 iOS Keychain 与 Android 安全凭证存储能力。 | 基础依赖；用于 Token、凭证和密钥；生物认证与设备安全策略按平台配置。 |
| [@op-engineering/op-sqlite](https://www.npmjs.com/package/@op-engineering/op-sqlite) | React Native SQLite 数据库，提供结构化数据查询、事务和原生数据库访问。 | 关系查询、大体量离线数据和本地事务业务；统一在 `core/storage` 封装，Feature Data 层定义数据映射与迁移，不用 MMKV 模拟关系数据库。 |

敏感信息不得因 MMKV 支持加密而直接写入普通 Store；加密密钥也不得与加密数据保存在同一普通配置中。

## 动画、手势与交互工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [react-native-reanimated](https://www.npmjs.com/package/react-native-reanimated) | 高性能动画与交互工具，可在 UI 运行时处理动画逻辑。 | 复杂动画基线；与 React Native 新架构、Worklets 版本成套匹配。 |
| [react-native-worklets](https://www.npmjs.com/package/react-native-worklets) | Reanimated 当前主版本所需的 Worklet 运行基础设施。 | 与 Reanimated 配套；Babel 插件和安装步骤遵循对应版本文档，不混用旧教程配置。 |
| [react-native-gesture-handler](https://www.npmjs.com/package/react-native-gesture-handler) | 原生手势识别和组合工具，提供拖拽、缩放及滑动等交互。 | 手势能力基线；根部配置 `GestureHandlerRootView`，处理与导航、列表的手势冲突。 |
| [@gorhom/bottom-sheet](https://www.npmjs.com/package/@gorhom/bottom-sheet) | 底部面板组件，支持吸附点、滚动内容和键盘交互。 | 底部面板功能；匹配 Reanimated、Gesture Handler，并按库文档配置 Provider。 |
| [lottie-react-native](https://www.npmjs.com/package/lottie-react-native) | Lottie 动画播放器，用于引导、加载状态与业务反馈。 | 设计动画功能；验证素材效果、播放性能及 React/React Native 最低版本。 |

## 列表与矢量图工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [@shopify/flash-list](https://www.npmjs.com/package/@shopify/flash-list) | Shopify 维护的高性能列表，通过视图回收处理长列表。 | 长列表能力；当前主版本面向新架构，业务组件不能依赖列表项永不复用的假设。 |
| [react-native-svg](https://www.npmjs.com/package/react-native-svg) | 在原生界面渲染 SVG 图形，提供矢量图标和绘图基础。 | 图标与矢量资源功能；直接导入 `.svg` 文件还需 Metro 转换配置，不能认为安装库就自动支持。 |

简单列表与普通图片使用 React Native 自带的 `FlatList`、`Image`。图片预加载、磁盘缓存和性能优化根据实际测量落实，不默认安装多个图片库。

## 文件、媒体与原生系统工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [react-native-image-picker](https://www.npmjs.com/package/react-native-image-picker) | 调用系统相册、相机入口选择或拍摄照片及视频。 | 媒体选择功能；不是实时相机帧处理或扫码引擎，URI 按平台规则处理。 |
| [@react-native-documents/picker](https://www.npmjs.com/package/@react-native-documents/picker) | 原生文档选择器，用于附件、文件导入和目录访问。 | 文件功能；使用当前包名，不沿用旧的 `react-native-document-picker` 安装示例；处理授权、URI 和文件生命周期。 |
| [react-native-permissions](https://www.npmjs.com/package/react-native-permissions) | 统一查询和申请相机、相册、定位等权限。 | 权限适配位于 `core/permissions`；区分拒绝、受限和永久拒绝，完成 Info.plist、Podfile 与 Manifest 配置。 |
| [react-native-webview](https://www.npmjs.com/package/react-native-webview) | 原生 WebView 容器，支持网页加载、导航控制和消息桥。 | H5 嵌入功能；限制可访问域，校验消息格式，避免将可信凭证暴露给任意页面。 |
| [react-native-video](https://www.npmjs.com/package/react-native-video) | 原生视频播放组件，支持播放控制、进度和平台媒体能力。 | 视频功能；编解码、DRM 和后台播放配置按平台及发行版本验证。 |
| [react-native-bootsplash](https://www.npmjs.com/package/react-native-bootsplash) | 管理原生启动屏与 React Native 首屏衔接。 | 启动体验基线；由 `app/bootstrap` 隐藏启动屏，不能把网络请求无限等待作为隐藏条件。 |

## 国际化与日期工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [i18next](https://www.npmjs.com/package/i18next) | 国际化核心，支持命名空间、插值、复数和语言回退。 | 多语言功能基线；文案按 Feature 命名空间维护。 |
| [react-i18next](https://www.npmjs.com/package/react-i18next) | i18next 的 React 绑定，提供翻译 Hook 和组件。 | 与 i18next 配套；界面使用文案 Key，不拼接多语言语句。 |
| [react-native-localize](https://www.npmjs.com/package/react-native-localize) | 获取系统语言、地区、时区等本地化信息。 | 原生语言感知；系统偏好与用户手动选择分别处理。 |
| [dayjs](https://www.npmjs.com/package/dayjs) | 轻量日期处理库，通过插件提供 UTC、时区和格式化能力。 | 日期业务；时区功能显式启用插件，业务时间与展示时间分开建模。 |

## 通知与推送工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [@notifee/react-native](https://www.npmjs.com/package/@notifee/react-native) | 本地通知展示、通知渠道、计划任务和通知交互管理。 | 通知展示功能；本身不是远程推送投递通道，点击事件集中适配后交给应用导航。 |
| [@react-native-firebase/app](https://www.npmjs.com/package/@react-native-firebase/app) | React Native Firebase 核心，初始化 Firebase 原生配置。 | 使用 Firebase 时引入；配套模块保持兼容发行版本。 |
| [@react-native-firebase/messaging](https://www.npmjs.com/package/@react-native-firebase/messaging) | FCM 远程推送客户端，处理设备 Token、前后台消息及消息接收。 | FCM 业务；与 Notifee 分工，统一通知点击入口避免重复处理；中国大陆 Android 设备仍需落实厂商通道和服务可达性。 |

## 商店订阅与付费工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [react-native-purchases](https://www.npmjs.com/package/react-native-purchases) | RevenueCat SDK，管理商店内购、订阅、恢复购买和用户权益。 | 订阅业务；涉及第三方服务及计费规则，不是通用支付 SDK；可信权益校验由服务端完成。 |

## 线上监控工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [@sentry/react-native](https://www.npmjs.com/package/@sentry/react-native) | 采集 JavaScript 异常、原生崩溃与性能遥测，支持导航追踪。 | 线上监控基线；在 `core/observability` 封装、启动阶段初始化；CI 上传 source map 和原生符号，脱敏用户数据。 |

## 自动化测试与质量工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [jest](https://www.npmjs.com/package/jest) | 单元测试、Mock、断言与测试执行框架。 | 开发依赖；验证 Domain、用例和数据映射；配置与 React Native 模板及测试库匹配。 |
| [@testing-library/react-native](https://www.npmjs.com/package/@testing-library/react-native) | 以用户可观察行为测试原生组件、表单、可访问性与交互。 | 开发依赖；测试组件行为，不依赖组件内部实现；测试渲染器按所选主版本安装。 |
| [detox](https://www.npmjs.com/package/detox) | 原生端到端测试工具，在 iOS 模拟器和 Android 模拟器等受支持环境运行流程。 | 开发依赖；测试放在 `e2e`，覆盖认证、深链和关键业务；使用专用测试构建及隔离账号。 |
| [typescript](https://www.npmjs.com/package/typescript) | 静态类型检查与类型系统。 | 开发依赖；启用严格检查，以 React Native 工程工具链兼容版本为准。 |
| [eslint](https://www.npmjs.com/package/eslint) | JavaScript/TypeScript 静态检查。 | 开发依赖；与 React Native ESLint 配置使用兼容版本。 |
| [@react-native/eslint-config](https://www.npmjs.com/package/@react-native/eslint-config) | React Native 官方维护的 ESLint 配置。 | 开发依赖；沿用对应 RN 工程基线，不独立追踪所有工具的最新主版本。 |
| [eslint-plugin-boundaries](https://www.npmjs.com/package/eslint-plugin-boundaries) | 检查目录层级与导入边界。 | 开发依赖；阻止 Domain 导入 SDK、Feature 深层互引及 Shared/Core 反向依赖。 |
| [prettier](https://www.npmjs.com/package/prettier) | 自动格式化代码，统一团队风格。 | 开发依赖；格式检查与 ESLint 分工，不用运行时依赖安装。 |

## 版本兼容与安装约束

以下版本仅是本次 npm 核查快照，用于展示真实依赖约束，不是可以直接整体复制安装的版本组合。

| 依赖关系 | 核查结果与执行要求 |
| --- | --- |
| React Native 新架构 | Reanimated 官方明确当前主版本仅支持新架构；FlashList 当前主版本同样以新架构为基础。旧工程升级先验证新架构，不通过回退至停止活跃维护的动画主版本解决新项目选型。 |
| MMKV 与 Nitro | 核查的 MMKV 为 `4.3.2`，peerDependencies 包含 `react-native-nitro-modules`；二者一起安装并验证原生构建。 |
| Reanimated 与 Worklets | 核查的 Reanimated `4.7.1` 声明 React Native `0.86 - 0.88` 与 Worklets `0.13.x`；以项目 RN 版本反查兼容表，不能对任意 RN 工程安装 latest。 |
| Lottie 与 React | 核查的 Lottie `7.5.0` 声明 React `>=19.2`、React Native `>=0.84`；旧项目必须选择兼容发行版。 |
| 文档选择器 | 核查的 `@react-native-documents/picker` `12.0.3` 要求 React Native `>=0.79.0`。 |
| 测试渲染器 | 核查的 Testing Library `14.1.0` 声明 React `>=19.0.0`、React Native `>=0.78`、`test-renderer ^1.0.0`；不得照搬旧版教程的渲染器配置。 |
| ESLint 工具链 | 核查的 `@react-native/eslint-config` `0.88.0` 声明 ESLint `^8.0.0` 或 `^9.0.0`，而 ESLint latest 为 `10.13.0`；项目采用满足配置约束的版本，不直接整体安装 latest。 |
| Firebase 模块 | 核查的 Firebase App 与 Messaging 均为 `27.0.0`，Messaging 对 App 声明精确配套版本；维护模块版本一致性。 |
| 原生联调 | 安装成功、TypeScript 通过不代表原生兼容；在目标 RN、Hermes、新架构与 iOS/Android Release 构建下验证实际功能。 |

## 与架构目录的对应关系

| 目录 | 放置能力 | 边界 |
| --- | --- | --- |
| `src/app/providers` | QueryClientProvider、安全区域、手势与其他根级 Provider | 只装配，不编写业务规则。 |
| `src/app/navigation` | React Navigation、Deep Link、认证区域与主区域导航 | 只声明入口，不实现 Screen 业务。 |
| `src/app/bootstrap` | 启动屏、监控、存储和原生 SDK 初始化编排 | 初始化失败可诊断，不阻塞启动至无限等待。 |
| `src/features/*/presentation` | Query 视图 Hook、表单、列表、动画与媒体 UI | 不直接操作 HTTP 客户端或原生存储。 |
| `src/features/*/application/state` | Zustand Feature Store | 只维护客户端状态，不复制服务端缓存。 |
| `src/features/*/domain` | 纯 TypeScript 实体、业务规则和 Repository 契约 | 不依赖 React、Zod、导航、状态库及原生 SDK。 |
| `src/features/*/data` | API、DTO、Zod 外部数据校验、Repository 实现、Query Key 与请求函数 | 转换为领域类型再对外返回，不泄漏 SDK 对象。 |
| `src/shared/ui`、`src/shared/theme` | 无业务基础组件、动画包装和设计 Token | 不导入 Feature。 |
| `src/core/api` | Axios 客户端、拦截器和错误标准化 | 不定义具体业务 Repository。 |
| `src/core/storage` | MMKV、Keychain 和 SQLite 基础适配 | Feature 定义业务 Key、模型与持久化时机。 |
| `src/core/native`、`src/core/permissions` | 文件、网络状态、通知、购买及权限适配 | 隐藏原生错误和平台差异。 |
| `src/core/observability` | Sentry 初始化、异常上报、脱敏 | 不在业务组件中散布 SDK 配置。 |
| `src/test`、`e2e` | 通用测试渲染器、Mock、固定数据和 Detox 用例 | 测试环境隔离生产账号及真实交易。 |

## 使用量核查记录

| 数据项 | 说明 |
| --- | --- |
| 下载统计区间 | 2026-10-12 至 2026-10-18，按 npm API 返回区间记录 |
| 数据来源 | npm Downloads API：`https://api.npmjs.org/downloads/point/last-week/<package>` |
| 元数据来源 | npm Registry：`https://registry.npmjs.org/<package>/latest`；本清单所列包的本次响应均未返回 npm 弃用标记 |
| 解释边界 | 周下载量不是 RN 独立用户数；配套依赖存在重复覆盖，不跨行相加为用户规模 |

| 第三方库 | 周下载次数 | 查询来源 |
| --- | --- | --- |
| `@react-navigation/native` | 3,664,977 | [npm API](https://api.npmjs.org/downloads/point/last-week/@react-navigation/native) |
| `react-native-screens` | 6,394,788 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-screens) |
| `react-native-safe-area-context` | 6,919,873 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-safe-area-context) |
| `@tanstack/react-query` | 40,112,338 | [npm API](https://api.npmjs.org/downloads/point/last-week/@tanstack/react-query) |
| `zustand` | 37,735,389 | [npm API](https://api.npmjs.org/downloads/point/last-week/zustand) |
| `axios` | 66,557,149 | [npm API](https://api.npmjs.org/downloads/point/last-week/axios) |
| `react-hook-form` | 32,173,754 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-hook-form) |
| `zod` | 207,048,213 | [npm API](https://api.npmjs.org/downloads/point/last-week/zod) |
| `react-native-mmkv` | 929,768 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-mmkv) |
| `react-native-keychain` | 253,957 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-keychain) |
| `@op-engineering/op-sqlite` | 417,985 | [npm API](https://api.npmjs.org/downloads/point/last-week/@op-engineering/op-sqlite) |
| `react-native-reanimated` | 6,063,921 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-reanimated) |
| `react-native-gesture-handler` | 5,886,001 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-gesture-handler) |
| `@gorhom/bottom-sheet` | 1,763,731 | [npm API](https://api.npmjs.org/downloads/point/last-week/@gorhom/bottom-sheet) |
| `@shopify/flash-list` | 1,517,090 | [npm API](https://api.npmjs.org/downloads/point/last-week/@shopify/flash-list) |
| `react-native-svg` | 5,390,874 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-svg) |
| `react-native-image-picker` | 252,632 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-image-picker) |
| `@react-native-documents/picker` | 179,555 | [npm API](https://api.npmjs.org/downloads/point/last-week/@react-native-documents/picker) |
| `react-native-webview` | 3,500,171 | [npm API](https://api.npmjs.org/downloads/point/last-week/react-native-webview) |
| `@notifee/react-native` | 202,115 | [npm API](https://api.npmjs.org/downloads/point/last-week/@notifee/react-native) |
| `@react-native-firebase/messaging` | 519,956 | [npm API](https://api.npmjs.org/downloads/point/last-week/@react-native-firebase/messaging) |
| `@sentry/react-native` | 2,430,122 | [npm API](https://api.npmjs.org/downloads/point/last-week/@sentry/react-native) |
| `@testing-library/react-native` | 2,651,296 | [npm API](https://api.npmjs.org/downloads/point/last-week/@testing-library/react-native) |
| `detox` | 250,233 | [npm API](https://api.npmjs.org/downloads/point/last-week/detox) |

## 社区维护核查记录

最近推送取 GitHub API 返回的 UTC 日期，不等于最新版本发布日期；所有记录均为本次快照。

| 仓库 | GitHub Star | 最近推送 | 是否归档 |
| --- | --- | --- | --- |
| [TanStack Query](https://github.com/TanStack/query) | 50,452 | 2026-10-21 | 否 |
| [Zustand](https://github.com/pmndrs/zustand) | 58,848 | 2026-10-14 | 否 |
| [Axios](https://github.com/axios/axios) | 109,303 | 2026-10-21 | 否 |
| [React Hook Form](https://github.com/react-hook-form/react-hook-form) | 44,873 | 2026-10-21 | 否 |
| [Zod](https://github.com/colinhacks/zod) | 44,166 | 2026-10-19 | 否 |
| [Nitro Modules](https://github.com/mrousavy/nitro) | 1,967 | 2026-10-21 | 否 |
| [React Native Keychain](https://github.com/oblador/react-native-keychain) | 3,485 | 2026-04-29 | 否 |
| [Reanimated](https://github.com/software-mansion/react-native-reanimated) | 11,032 | 2026-10-21 | 否 |
| [Gesture Handler](https://github.com/software-mansion/react-native-gesture-handler) | 6,793 | 2026-10-21 | 否 |
| [Screens](https://github.com/software-mansion/react-native-screens) | 3,740 | 2026-10-21 | 否 |
| [Safe Area Context](https://github.com/AppAndFlow/react-native-safe-area-context) | 2,768 | 2026-10-21 | 否 |
| [Bottom Sheet](https://github.com/gorhom/react-native-bottom-sheet) | 9,109 | 2026-05-09 | 否 |
| [FlashList](https://github.com/Shopify/flash-list) | 7,243 | 2026-10-01 | 否 |
| [Documents Picker](https://github.com/react-native-documents/document-picker) | 1,515 | 2026-10-20 | 否 |
| [React Native Firebase](https://github.com/invertase/react-native-firebase) | 12,319 | 2026-10-21 | 否 |
| [Sentry React Native](https://github.com/getsentry/sentry-react-native) | 1,824 | 2026-10-21 | 否 |

## 官方资料与兼容性依据

| 来源 | 核查结论 | 本清单落地 |
| --- | --- | --- |
| [React Native：Using Libraries](https://reactnative.dev/docs/libraries) | 官方通过 React Native Directory 查找生态库；原生依赖需要重新构建应用。 | 库选型与原生集成一并检查，不把 JS 安装成功当作集成完成。 |
| [React Navigation 文档](https://reactnavigation.org/docs/getting-started/) | 导航需要配套原生基础依赖及平台配置。 | 固定 React Navigation，统一导航目录和类型。 |
| [TanStack Query：React Native](https://tanstack.com/query/latest/docs/framework/react/react-native) | 原生网络和前后台状态通过 `onlineManager`、`focusManager` 适配。 | App Providers 统一装配 NetInfo 与 AppState 监听并清理订阅。 |
| [Reanimated 兼容表](https://docs.swmansion.com/react-native-reanimated/docs/guides/compatibility/) | 当前主版本要求新架构，React Native 与 Worklets 存在版本匹配约束。 | 安装前以项目 RN 版本查兼容矩阵。 |
| [FlashList 文档](https://shopify.github.io/flash-list/docs/) | 当前主版本面向 React Native 新架构。 | 长列表选型以新架构为基线。 |
| [MMKV 仓库](https://github.com/mrousavy/react-native-mmkv) | 当前 npm 元数据声明 Nitro Modules peerDependency。 | MMKV 与 Nitro 成套装配，不复用旧版安装步骤。 |

本清单优先固定核心框架和边界清晰的成熟能力库。新增 SDK 必须说明它补齐的业务能力、所属目录、原生兼容范围和维护依据；不因功能清单变长就扩大工程依赖。
