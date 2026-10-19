# KMP 项目的常用第三方库

本清单面向 Android、iOS、Desktop 三端的中大型 Kotlin Multiplatform 项目，与 [KMP 架构规范](kmp-架构.md) 保持一致。库选择固定围绕 Compose Multiplatform、Koin、Ktor、Room KMP 和 Gradle 多模块架构，不引入功能重叠的替代方案。

# Kotlin 基础与并发工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `kotlinx-coroutines-core` | Kotlin 官方协程库，用于跨端异步任务、并发控制、`Flow` 与 `StateFlow` 状态流，是 ViewModel、Use Case 和数据层的基础。 |
| `kotlinx-serialization-json` | Kotlin 官方序列化库，用于 API JSON 编解码、持久化对象序列化和类型安全路由参数。 |
| `kotlinx-datetime` | Kotlin 官方跨平台日期时间库，用于处理时区、日期、时间与时间戳，避免在共享层依赖 Java 或 Apple 平台日期 API。 |
| `kotlinx-collections-immutable` | Kotlin 官方不可变集合库，用于 `UiState`、领域模型和状态流，避免可变集合导致界面状态被意外修改。 |
| `kotlinx-atomicfu` | Kotlin 官方原子操作库，用于跨平台并发场景中的轻量原子读写和状态控制。 |

# 网络通信工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `io.ktor:ktor-client-core` | Ktor 跨平台 HTTP 客户端核心，用于 REST API 请求、请求管线、超时、重试与统一错误处理。 |
| `io.ktor:ktor-client-content-negotiation` | Ktor 内容协商插件，与 `kotlinx-serialization-json` 结合，实现 HTTP 请求和响应的 JSON 自动编解码。 |
| `io.ktor:ktor-client-logging` | Ktor 请求日志插件，用于开发和排障阶段记录 URL、状态码、耗时与请求错误。 |
| `io.ktor:ktor-client-websockets` | Ktor WebSocket 客户端，用于 IM、实时通知、协作编辑和 AI 流式事件等双向通信场景。 |
| `io.ktor:ktor-client-okhttp` | Android 平台 Ktor 引擎，基于 OkHttp 执行 HTTP 与 WebSocket 请求。 |
| `io.ktor:ktor-client-darwin` | iOS 平台 Ktor 引擎，基于 Darwin 网络栈执行 HTTP 与 WebSocket 请求。 |
| `io.ktor:ktor-client-cio` | Desktop JVM 平台 Ktor 引擎，用于桌面端 HTTP 与 WebSocket 请求。 |

# 依赖注入工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `io.insert-koin:koin-core` | Koin 跨平台依赖注入核心，用于注册 Core Module、Feature Module、Repository、Use Case 与 ViewModel。 |
| `io.insert-koin:koin-compose` | Koin 与 Compose Multiplatform 集成库，用于在共享 Compose UI 中按生命周期获取 ViewModel 和依赖。 |
| `io.insert-koin:koin-test` | Koin 测试库，用于校验 Module 定义、依赖图和注入实例。 |

# 本地存储工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `androidx.room:room-runtime`（Room KMP） | AndroidX 跨平台关系型数据库框架，用于实体建模、DAO 查询、Migration 和离线缓存；共享层定义数据库模型与 DAO。 |
| `androidx.room:room-compiler` | Room KSP 代码生成器，用于生成类型安全的数据库访问实现。 |
| `androidx.room:room-sqlite-bundled` | Room 的跨平台 SQLite Driver，用于 iOS 与 Desktop 的一致数据库运行时。 |
| `androidx.datastore:datastore-preferences`（DataStore KMP） | AndroidX 跨平台键值存储库，用于应用偏好、功能开关、引导状态和非敏感轻量配置。 |
| `com.squareup.okio:okio` | Square 跨平台 I/O 库，用于文件读写、流处理、二进制数据和文件系统抽象。 |

# 界面布局与图片工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `org.jetbrains.compose`（Compose Multiplatform） | JetBrains 跨平台声明式 UI 框架，用于共享 Android、iOS 与 Desktop 的界面、主题和资源。 |
| `org.jetbrains.androidx.navigation:navigation-compose` | Compose Multiplatform 导航库，用于类型安全路由、嵌套导航图、返回栈和 Deep Link。 |
| `androidx.lifecycle:lifecycle-viewmodel-compose` | Kotlin Multiplatform ViewModel 与 Compose 集成库，用于在共享 UI 中管理生命周期绑定的 ViewModel 与界面状态。 |
| `io.coil-kt.coil3:coil-compose` | Coil 3 跨平台图片加载库，支持 Android、iOS 和 JVM；提供异步加载、内存与磁盘缓存、占位图和 Compose `AsyncImage`。 |

# 动画与交互组件

| 第三方库 | 简要介绍 |
| --- | --- |
| `io.github.alexzhirkevich:compottie` | Compose Multiplatform 的 Lottie 动画播放器，用于引导页、加载状态、空状态和业务反馈动画。 |

# 监控与性能工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `co.touchlab:kermit` | Touchlab 的跨平台日志库，用于统一 Android、iOS 与 Desktop 的分级日志、Tag 和平台日志输出。 |

# 开发与代码检查工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `io.gitlab.arturbosch.detekt` | Kotlin 静态分析工具，用于检查复杂度、潜在缺陷、重复代码和团队代码规范。 |
| `com.autonomousapps.dependency-analysis` | Gradle 依赖分析插件，用于识别未使用、错误暴露和冗余的模块或三方依赖，维护多模块依赖边界。 |
| `com.google.devtools.ksp` | Kotlin Symbol Processing 代码生成工具，供 Room、Koin 等库生成类型安全实现，替代反射式运行时处理。 |

# 自动化测试工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `kotlin-test` | Kotlin 官方跨平台测试框架，用于 `commonTest` 中的断言、测试组织和跨端单元测试。 |
| `kotlinx-coroutines-test` | Kotlin 官方协程测试库，用于控制测试调度器、虚拟时间和 `Flow` 行为，验证 ViewModel 与 Use Case。 |
| `app.cash.turbine:turbine` | Cash App 的 Flow 测试库，用于断言 `StateFlow`、`SharedFlow` 的发射顺序、完成和异常。 |
| `org.jetbrains.compose.ui:ui-test` | Compose Multiplatform 跨端 UI 测试库，用于验证 Screen 语义节点、用户交互、加载态、空态和错误态。 |
