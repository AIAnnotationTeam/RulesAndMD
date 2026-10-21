# Android 项目的常用第三方库

本清单面向 Kotlin、Jetpack Compose、Gradle 多模块的中大型 Android 项目，与 [Android 架构规范](android-架构.md) 保持一致。固定使用 Hilt、Retrofit、OkHttp、kotlinx.serialization、Room、DataStore、Paging、WorkManager 和 Coil，不同时引入职责重叠的基础框架。

AndroidX 和 Kotlin 官方生态组件承担工程基础能力，社区库补齐网络、图片、动画、监控和工程效率。下文明确维护来源，不把官方组件误称为独立社区项目。业务扩展仅在功能存在时安装，清单不是全量安装套餐。

## 选型与核查口径

| 项目 | 口径 |
| --- | --- |
| 核查日期 | 2026-10-21 |
| 适用范围 | Kotlin、Compose、Android 原生应用、Gradle 多模块；不以 XML View、Java 或 KMP 作为模板基线 |
| 使用规模 | 结合 Android 官方生态地位、成熟项目采用情况和 GitHub 关注度判断；没有可统一核验的 Maven 独立用户数，不虚构用户量、下载量或排名 |
| 社区维护 | 核查 GitHub 归档状态与最近推送记录；推送不等于版本发布，机器人更新不单独证明维护质量 |
| 指标边界 | Star 是仓库关注度，不是生产用户数；AndroidX 等多组件仓库不能把总 Star 当成单个组件的使用量 |
| 信息限制 | 部分 Android Developers 页面访问超时；已通过 AndroidX 官方源码、组件 README 和成功访问的官方文档交叉核查相关集成细节 |
| 版本来源 | 所有版本集中于 `gradle/libs.versions.toml`，约束由 Convention Plugin 统一应用；不使用动态版本或 SNAPSHOT 作为生产基线 |
| 依赖范围 | 默认 `implementation`；处理器使用 `ksp`；单测使用 `testImplementation`；设备测试使用 `androidTestImplementation`；调试工具使用 `debugImplementation` |

## Kotlin 基础与异步工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `org.jetbrains.kotlinx:kotlinx-coroutines-android` | JetBrains Kotlin | Android 协程与主线程调度支持；ViewModel 使用结构化并发，禁止以 `GlobalScope` 承载业务任务。 |
| `org.jetbrains.kotlinx:kotlinx-coroutines-core` | JetBrains Kotlin | 协程、Flow、StateFlow 和并发基础；纯 Kotlin Domain 仅按需要依赖该模块，不引入 Android 协程模块。 |
| `org.jetbrains.kotlinx:kotlinx-serialization-json` | JetBrains Kotlin | Kotlin JSON 编解码；用于 Data 层 DTO，序列化结构不向 UI 或 Domain 泄漏。 |

## UI、状态与导航组件

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `androidx.compose:compose-bom` | AndroidX | Compose 组件版本对齐平台；通过 Gradle `platform(...)` 引入，不是运行时 UI 库，也不管理所有 AndroidX 版本。 |
| `androidx.compose.ui:ui` | AndroidX | Compose UI 基础；页面、组件按 Feature 和 Design System 归属。 |
| `androidx.compose.foundation:foundation` | AndroidX | 布局、滚动、列表、分页容器及基础交互；SDK 已覆盖的 UI 能力不重复引入旧扩展库。 |
| `androidx.compose.material3:material3` | AndroidX | Material 3 组件和主题；统一封装于 `:core:designsystem`。 |
| `androidx.activity:activity-compose` | AndroidX | Activity 与 Compose 集成、Activity Result 等宿主能力；平台行为不直接进入 Domain。 |
| `androidx.lifecycle:lifecycle-viewmodel-compose` | AndroidX | Compose 与 ViewModel 生命周期集成；ViewModel 归属对应 Feature。 |
| `androidx.lifecycle:lifecycle-runtime-compose` | AndroidX | 提供 `collectAsStateWithLifecycle` 等生命周期感知状态收集。 |
| `androidx.navigation:navigation-compose` | AndroidX | Compose 页面导航、返回栈与深链；根导航由 `:app` 编排，类型安全路由使用 Kotlin Serialization。 |

## 依赖注入工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `com.google.dagger:hilt-android` | Google Dagger/Hilt | Android 依赖注入；优先构造函数注入，以 Hilt Module 绑定 Repository 接口与实现。 |
| `com.google.dagger:hilt-compiler` | Google Dagger/Hilt | Hilt 代码生成处理器；通过 KSP 引入，不进入运行依赖。 |
| `androidx.hilt:hilt-lifecycle-viewmodel-compose` | AndroidX | 在 Compose 中获取 Hilt ViewModel；采用该模块提供的生命周期集成，按导航返回栈约束 ViewModel 作用域。 |
| `androidx.hilt:hilt-work` | AndroidX | Hilt 与 WorkManager 集成，给 Worker 注入依赖；存在后台任务时安装。 |
| `androidx.hilt:hilt-compiler` | AndroidX | AndroidX Hilt 扩展的处理器，用于 `@HiltWorker` 等集成；与 Dagger Hilt 编译器职责不同。 |

## 网络通信与数据解析工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `com.squareup.retrofit2:retrofit` | Retrofit 维护社区 | 声明式 HTTP API，支持挂起函数；API 和 DTO 放 Feature Data，客户端配置放 `:core:network`。 |
| `com.squareup.okhttp3:okhttp` | OkHttp 维护社区 | HTTP、连接池、缓存、WebSocket 和网络拦截器；统一配置超时、认证与错误处理。 |
| `com.squareup.retrofit2:converter-kotlinx-serialization` | Retrofit 维护社区 | Retrofit 官方维护的 Kotlin Serialization 转换器；不使用历史独立仓库中的旧转换器坐标。 |
| `com.squareup.okhttp3:logging-interceptor` | OkHttp 维护社区 | 请求、响应与网络排障日志；仅调试环境启用详细正文，脱敏 Token、Cookie 和个人信息。 |

网络库不承担业务规则。Repository 协调缓存与数据源，Mapper 将 DTO 转为 Domain Model。OkHttp 已提供 WebSocket，普通双向通信不额外安装另一套底层客户端。

## 本地存储工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `androidx.room:room-runtime` | AndroidX | SQLite 数据库抽象，提供实体、DAO、事务与数据库生命周期管理；用于结构化数据和离线缓存。 |
| `androidx.room:room-compiler` | AndroidX | 生成 Room DAO 和数据库实现，使用 KSP；Schema 导出纳入版本控制。 |
| `androidx.datastore:datastore-preferences` | AndroidX | 基于协程与 Flow 的偏好存储；保存主题、语言、引导标记等轻量配置，不替代关系数据库。 |

### 敏感信息与安全边界

| 能力 | 固定实现与边界 |
| --- | --- |
| 密钥管理 | 使用 Android SDK 的 Android Keystore，封装于 `:core:datastore`；不是第三方库。 |
| 凭证持久化 | Keystore 管理加密密钥；使用平台密码学 API 加密凭证后保存密文，处理设备策略、密钥失效、备份和登出清理；不宣称 Keystore 可以直接存放任意业务数据。 |
| DataStore 安全属性 | DataStore 默认不等于加密存储；不得直接保存明文 Token、密码或私钥。 |
| 历史 Security Crypto 封装 | 本次官方源码核实 `EncryptedSharedPreferences` 类已标记弃用；新项目不把该类当作长期安全基线。弃用说明不代表可以将敏感信息改为明文存储。 |

## 分页与后台任务工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `androidx.paging:paging-runtime` | AndroidX | 分页数据加载、重试和缓存；PagingSource、RemoteMediator 放 Feature Data 层。 |
| `androidx.paging:paging-compose` | AndroidX | 将 Paging 数据接入 Compose 列表；页面负责展示加载、错误与重试状态。 |
| `androidx.work:work-runtime` | AndroidX | 约束驱动、可持久化的后台工作，当前运行库包含协程 Worker 能力；Worker 放所属 Feature Data，处理幂等和重试。 |

WorkManager 不用于精确闹钟、永久在线连接或必须立即完成的操作。前台服务、精确闹钟和推送分别遵循 Android 平台限制，不用换个库名绕过系统规则。

## 图片与动画工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `io.coil-kt.coil3:coil-compose` | Coil 社区 | Compose 图片加载，提供 AsyncImage、缓存和图片解码；统一接入设计系统图片组件。 |
| `io.coil-kt.coil3:coil-network-okhttp` | Coil 社区 | Coil 的 OkHttp 网络图片实现；Android 网络图片场景与 `coil-compose` 配套安装。 |
| `com.airbnb.android:lottie-compose` | Airbnb/Lottie 社区 | Compose 中播放 Lottie 动画，用于引导、加载和业务反馈；控制素材体积与并发播放数量。 |

普通位移、渐变、过渡和手势动画使用 Compose 内置能力，不因 Lottie 覆盖设计动画就替换基础动画 API。

## 音视频与相机组件

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `androidx.media3:media3-exoplayer` | AndroidX | 音视频播放引擎，支持流媒体、播放列表和平台媒体能力；格式、DRM 与设备解码能力分别验证。 |
| `androidx.media3:media3-ui-compose` | AndroidX | Media3 的 Compose 播放界面与状态集成；与 Media3 其他模块使用相同发行版本。 |
| `androidx.media3:media3-session` | AndroidX | 媒体会话、系统播放控制与媒体服务集成；存在后台播放时引入并完成服务配置。 |
| `androidx.camera:camera-camera2` | AndroidX CameraX | CameraX 的 Camera2 后端，简化设备相机兼容。 |
| `androidx.camera:camera-lifecycle` | AndroidX CameraX | 把相机用例绑定到生命周期；权限与资源释放由平台适配协调。 |
| `androidx.camera:camera-view` | AndroidX CameraX | 提供 PreviewView；本模板通过 Compose `AndroidView` 封装相机预览。 |

系统照片选择、文件选择和简单拍摄使用 Activity Result API，不为系统已经提供的选择器再叠加第三方选择器框架。

## 推送与商店计费组件

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `com.google.firebase:firebase-messaging` | Google Firebase | FCM 消息接收与设备 Token 管理；与通知展示分工，不等同于中国大陆 Android 全厂商推送通道。 |
| `com.android.billingclient:billing-ktx` | Google Play | Google Play 应用内购买与订阅；商品、购买确认和可信权益校验经服务端协调，不覆盖国内所有应用商店或通用支付。 |

仅有对应发行渠道和业务需求时引入这些 SDK。中国大陆分发需要单独落实厂商服务及可达性，渠道 SDK 的差异封装在基础适配中，不侵入 Domain。

## 日志、监控与调试工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `com.jakewharton.timber:timber` | Timber 社区 | 分级日志和可插拔日志树，统一 Tag 与调试输出；生产日志策略由 `:core:analytics` 控制。 |
| `io.sentry:sentry-android` | Sentry | 线上崩溃、ANR 与性能遥测；经 `:core:analytics` 封装，上传混淆映射及原生符号，控制采样与隐私数据。 |
| `com.squareup.leakcanary:leakcanary-android` | LeakCanary 社区 | 自动定位调试环境的内存泄漏；只使用 `debugImplementation`，不打进 Release。 |

Timber 服务日志输出，Sentry 服务集中诊断，LeakCanary 服务开发阶段内存排障，三者不互相替代。

## 单元测试与集成测试工具

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `org.junit.jupiter:junit-jupiter` | JUnit 团队 | 沿用架构模板的 JUnit 5 JVM 单元测试基线；配置 `useJUnitPlatform()`，不把 JVM 运行器直接用于 Android 设备测试。 |
| `org.jetbrains.kotlinx:kotlinx-coroutines-test` | JetBrains Kotlin | 测试调度器、虚拟时间和 `runTest`；控制 ViewModel、UseCase 的异步行为。 |
| `app.cash.turbine:turbine` | Cash App | 断言 Flow/StateFlow 的事件、完成与异常；用于状态流测试。 |
| `io.mockk:mockk` | MockK 社区 | Kotlin Mock，支持挂起函数；使用于 JVM 测试，能用简单 Fake 表达的业务依赖保持 Fake。 |
| `com.squareup.okhttp3:mockwebserver3` | OkHttp 维护社区 | 本地可控 HTTP 服务，测试响应、请求内容、错误和网络时序；与 OkHttp 对齐发行版本。 |
| `androidx.room:room-testing` | AndroidX | 验证 Room Schema 和迁移；数据库升级不能只测试新建库。 |
| `androidx.work:work-testing` | AndroidX | 测试 Worker 和约束触发；验证重试与幂等。 |
| `com.google.dagger:hilt-android-testing` | Google Dagger/Hilt | 为 Hilt 测试装配依赖与替换绑定；处理器配置覆盖对应测试 Source Set。 |

### Android 设备测试与性能组件

| 库与坐标 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| `androidx.test:runner` | AndroidX Test | AndroidJUnitRunner 设备测试基础；与 JVM JUnit Platform 分开配置。 |
| `androidx.test.ext:junit` | AndroidX Test | AndroidJUnit4 测试扩展，服务 Android 集成与设备测试。 |
| `androidx.compose.ui:ui-test-junit4` | AndroidX Compose | Compose 语义节点、交互和可访问性测试；采用 AndroidJUnit4 设备测试配置。 |
| `androidx.compose.ui:ui-test-manifest` | AndroidX Compose | 为相应 Compose 测试提供宿主配置，使用 `debugImplementation`。 |
| `androidx.test.uiautomator:uiautomator` | AndroidX Test | 验证系统权限弹窗、通知等跨应用 UI 流程。 |
| `androidx.benchmark:benchmark-macro-junit4` | AndroidX Benchmark | 测量启动、滚动和关键交互性能；放在 `:benchmark`，针对可分析的 Release 类构建。 |
| `androidx.baselineprofile`（Gradle 插件） | AndroidX Benchmark | 生成和集成 Baseline Profile；性能收益用基准测试验证。 |

## 构建、代码生成与质量工具

| 工具 | 维护来源 | 简要介绍与使用边界 |
| --- | --- | --- |
| KSP：`com.google.devtools.ksp` | Google/Kotlin 生态 | Kotlin 符号处理插件，供 Room、Hilt 生成实现；不把编译处理器打进 APK。 |
| Kotlin Compose 插件：`org.jetbrains.kotlin.plugin.compose` | JetBrains Kotlin | 配置 Compose 编译器，与所用 Kotlin 版本保持一致。 |
| Kotlin Serialization 插件：`org.jetbrains.kotlin.plugin.serialization` | JetBrains Kotlin | 生成序列化代码，与 Kotlin 编译器版本保持一致。 |
| Hilt 插件：`com.google.dagger.hilt.android` | Google Dagger/Hilt | Hilt 构建集成，与 Hilt 运行库和编译器对齐。 |
| Room 插件：`androidx.room` | AndroidX | 管理 Room Schema 输出等构建配置，与 Room 组件对齐。 |
| [Detekt](https://detekt.dev/) | Detekt 社区 | Kotlin 静态分析，检查复杂度、潜在缺陷和代码规范；规则放 `config`，不启用与格式工具重复的规则。 |
| [ktlint](https://github.com/pinterest/ktlint) | ktlint 社区 | Kotlin 格式规范检查；Gradle 集成使用 `org.jlleitschuh.gradle.ktlint` 插件，统一格式规则。 |
| Gradle 插件：`com.autonomousapps.dependency-analysis` | Autonomous Apps | 检查多模块未使用依赖及 `api`/`implementation` 配置，辅助维护公开 API；模块允许依赖方向还需单独的构建规则约束。 |

Android Lint、Gradle Version Catalog、Convention Plugin 和 R8 属于官方工程工具链，不冒充额外第三方运行库。

## 兼容性与安装约束

| 项目 | 执行要求 |
| --- | --- |
| Compose BOM | 只对齐其覆盖的 Compose 组件；不把 Kotlin、Hilt、Navigation、Room、Media3 和 Compose 编译器交给 BOM 管理。 |
| Kotlin 与代码生成 | Kotlin、Compose 编译器、Serialization 插件、KSP、AGP 和 Gradle 按兼容矩阵升级，不假设所有版本号都相同。 |
| Room 与 Hilt | 使用各自支持的 KSP 配置；同一处理器不同时用 KAPT 和 KSP 重复生成。 |
| Coil 网络图片 | Compose 图片模块不自动代表具备网络加载实现；本模板配套引入 `coil-network-okhttp`。 |
| Retrofit 转换器 | 使用 `com.squareup.retrofit2:converter-kotlinx-serialization`，不沿用旧独立转换器。 |
| MockWebServer | 本模板使用 `mockwebserver3` API，与 OkHttp 配套；不混用旧包名的测试代码。 |
| Media3 | ExoPlayer、UI、Session 等组件对齐发行版本；不再为新项目引入旧的 `com.google.android.exoplayer` 系列。 |
| JUnit | JVM 单元测试沿用 JUnit 5；Android 设备测试使用 AndroidJUnitRunner、AndroidJUnit4 和 Compose UI Test，不宣称 JUnit 5 原生覆盖所有测试。 |
| Android 平台 | 库升级核对最低 API、compileSdk、JDK、权限、Manifest 与 R8 配置；通过 Release 构建后再验证运行功能。 |

## 与架构模块的对应关系

| 模块或层级 | 第三方能力 | 边界 |
| --- | --- | --- |
| `:app` | Hilt Application、根导航、SDK 启动装配 | 编排流程，不实现业务规则。 |
| Feature `presentation` | Compose、ViewModel、Lifecycle、Paging UI、Coil 与 Lottie | 收集状态、发送事件；不调用数据库或 HTTP 客户端。 |
| Feature `domain` | Kotlin 和按需使用的 Coroutines Core | 不依赖 Android SDK、Hilt、Room、Retrofit 和 Compose。 |
| Feature `data` | Retrofit API、序列化 DTO、Room Entity/DAO、PagingSource、Worker | 实现 Repository，向上返回领域模型。 |
| `:core:network` | Retrofit、OkHttp、转换器、网络错误适配 | 不依赖具体 Feature。 |
| `:core:database` | Room 基础配置、Schema 和迁移基础设施 | 与 Feature 数据归属统一设计，禁止 Core 反向导入 Feature。 |
| `:core:datastore` | DataStore、Keystore 与密文存储适配 | 统一凭证生命周期和配置存储。 |
| `:core:designsystem`、`:core:ui` | Material、通用图片及动画组件 | 不容纳业务专属组件。 |
| `:core:analytics` | Timber、Sentry、可观测性适配 | 脱敏与环境开关集中配置。 |
| `:core:testing` | Fixture、Fake、协程测试与公共断言 | 测试依赖不传递进生产 APK。 |
| `:benchmark`、`e2e` | Macrobenchmark、UI Automator、Compose 测试 | 分别验证性能与用户流程。 |
| `build-logic` | 质量规则、编译器插件、依赖约束 | 统一配置，不让每个 Feature 自行维护一套版本。 |

## 社区维护核查记录

以下为本次实际取得的 GitHub 仓库快照，最近推送按 API 返回的 UTC 日期记录。Star 仅作生态关注度参考；成熟库更新频率不同，不用单一频率指标淘汰稳定基础库。

| 项目与来源 | GitHub Star | 最近推送 | 是否归档 |
| --- | --- | --- | --- |
| [Retrofit](https://github.com/square/retrofit) | 43,934 | 2026-10-20 | 否 |
| [OkHttp](https://github.com/square/okhttp) | 47,091 | 2026-10-21 | 否 |
| [Kotlin Coroutines](https://github.com/Kotlin/kotlinx.coroutines) | 13,825 | 2026-10-19 | 否 |
| [Kotlin Serialization](https://github.com/Kotlin/kotlinx.serialization) | 5,949 | 2026-10-20 | 否 |
| [Dagger/Hilt](https://github.com/google/dagger) | 17,691 | 2026-10-20 | 否 |
| [Coil](https://github.com/coil-kt/coil) | 11,945 | 2026-10-21 | 否 |
| [Lottie Android](https://github.com/airbnb/lottie-android) | 35,752 | 2026-02-15 | 否 |
| [Timber](https://github.com/JakeWharton/timber) | 10,850 | 2026-10-21 | 否 |
| [LeakCanary](https://github.com/square/leakcanary) | 30,001 | 2026-10-16 | 否 |
| [Turbine](https://github.com/cashapp/turbine) | 2,868 | 2026-10-20 | 否 |
| [MockK](https://github.com/mockk/mockk) | 5,763 | 2026-10-20 | 否 |
| [JUnit](https://github.com/junit-team/junit-framework) | 7,058 | 2026-10-21 | 否 |
| [Sentry Java/Android](https://github.com/getsentry/sentry-java) | 1,357 | 2026-10-21 | 否 |
| [Detekt](https://github.com/detekt/detekt) | 7,099 | 2026-10-21 | 否 |
| [ktlint](https://github.com/pinterest/ktlint) | 6,766 | 2026-10-20 | 否 |
| [Dependency Analysis](https://github.com/autonomousapps/dependency-analysis-gradle-plugin) | 2,187 | 2026-10-19 | 否 |
| [KSP](https://github.com/google/ksp) | 3,485 | 2026-10-21 | 否 |
| [AndroidX](https://github.com/androidx/androidx) | 6,114 | 2026-10-21 | 否 |
| [Media3](https://github.com/androidx/media) | 3,052 | 2026-10-21 | 否 |
| [Now in Android 官方示例](https://github.com/android/nowinandroid) | 21,950 | 2026-10-21 | 否 |

## 资料与集成依据

| 来源 | 用途 |
| --- | --- |
| [Android 架构指南](https://developer.android.com/topic/architecture) | 分层、单向数据流及 Repository 的官方参考。 |
| [Now in Android](https://github.com/android/nowinandroid) | 中大型 Compose、多模块与可测试数据层的官方工程参考；示例项目本身不是运行依赖。 |
| [AndroidX Hilt Compose 模块源码](https://github.com/androidx/androidx/tree/androidx-main/hilt/hilt-lifecycle-viewmodel-compose) | 本次核实 Compose ViewModel 集成模块的归属与构建配置。 |
| [Retrofit Kotlin Serialization 转换器](https://github.com/square/retrofit/tree/trunk/retrofit-converters/kotlinx-serialization) | 本次核实转换器 Maven 坐标。 |
| [Coil Getting Started](https://coil-kt.github.io/coil/getting_started/) | 本次核实 Compose 与网络图片配套依赖。 |
| [MockWebServer README](https://github.com/square/okhttp/tree/main/mockwebserver) | 本次核实 `mockwebserver3` 坐标。 |
| [Media3 Compose README](https://github.com/androidx/media/tree/release/libraries/ui_compose) | 本次核实 `media3-ui-compose` 坐标。 |
| [EncryptedSharedPreferences 官方源码](https://github.com/androidx/androidx/blob/androidx-main/security/security-crypto/src/main/java/androidx/security/crypto/EncryptedSharedPreferences.java) | 本次核实该类的弃用标记；安全存储保持 Keystore 与平台加密 API 边界。 |

新增依赖必须说明其业务能力、所属模块、维护依据与兼容约束。工程基线保持统一，扩展能力围绕实际业务增长，避免通过增加框架数量制造新的维护成本。
