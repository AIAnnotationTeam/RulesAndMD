# Android 中大型项目架构规范

## 第三方配置管理（强制）

第三方服务地址、应用 ID、API Key 等项目配置统一定义在 `core/common/src/main/kotlin/com/company/core/common/config/AppEnvironment.kt` 的 `AppEnvironment` 中，其他代码统一引用，禁止分散定义；本工程不强制使用环境文件。

本模板面向使用 Kotlin、Jetpack Compose 和 Gradle 多模块的中大型 Android 项目。固定采用 Android 官方架构原则中的 UI 层 + Data 层 + 可复用 Domain 层，并按业务 Feature 拆分模块。

固定技术栈：Jetpack Compose、Material 3、Navigation Compose、ViewModel、StateFlow、Kotlin Coroutines、Hilt、Retrofit、OkHttp、kotlinx.serialization、Room、DataStore、Paging 3、WorkManager、Coil。

## 1. 架构层级

| 层级 | Gradle 模块 | 职责 |
| --- | --- | --- |
| Application | `:app` | 应用启动、全局依赖注入、根导航、系统配置和跨 Feature 编排 |
| Feature Presentation | `:feature:<name>` 的 `presentation` | Compose Screen、ViewModel、UiState、UiEvent、页面导航 |
| Feature Domain | `:feature:<name>` 的 `domain` | 业务实体、UseCase、Repository 接口和业务规则 |
| Feature Data | `:feature:<name>` 的 `data` | API、DTO、数据库实体、Mapper、Repository 实现和缓存策略 |
| Core UI | `:core:designsystem`、`:core:ui` | 主题、Design Token、无业务语义的通用组件 |
| Core Data | `:core:network`、`:core:database`、`:core:datastore` | 网络、数据库、键值存储等技术基础能力 |
| Core Common | `:core:common`、`:core:model`、`:core:testing` | 通用错误、调度器、稳定共享模型和测试工具 |

模块依赖方向固定为：

```text
:app → :feature:* → :core:*
:feature:<name>:presentation → :feature:<name>:domain
:feature:<name>:data → :feature:<name>:domain
:feature:<name>:data → :core:network、:core:database、:core:datastore
:core:* → 更基础的 :core:*
```

Feature 之间禁止直接依赖。跨 Feature 流程由 `:app` 编排；跨 Feature 稳定契约下沉到 `:core:model`；Data 与 Domain 的实现绑定由 Hilt Module 完成。

## 2. 固定技术配置

| 能力 | 固定配置 |
| --- | --- |
| UI | Jetpack Compose + Material 3 |
| 页面状态 | ViewModel + `StateFlow<UiState>` |
| 事件流 | 单向数据流：State 向下、Event 向上 |
| 导航 | Navigation Compose；路由使用类型安全参数封装 |
| 异步 | Kotlin Coroutines + Flow；ViewModel 使用 `viewModelScope` |
| 依赖注入 | Hilt；使用构造函数注入，禁止 Service Locator |
| 网络 | Retrofit + OkHttp + kotlinx.serialization |
| 数据库 | Room + KSP；Repository 负责远端与本地协调 |
| 键值存储 | Jetpack DataStore；敏感信息使用 Android Keystore 封装 |
| 分页 | Paging 3；分页源放在 Feature Data |
| 后台任务 | WorkManager；任务定义放对应 Feature Data |
| 图片 | Coil Compose |
| 构建 | Gradle Version Catalog + Convention Plugin |
| 测试 | JUnit 5、Turbine、Compose UI Test、MockWebServer、Macrobenchmark |

## 2.1 默认第三方库基线

| 能力 | 默认库 | 固定归属 |
| --- | --- | --- |
| UI 与导航 | Jetpack Compose、Material 3、Navigation Compose、Lifecycle Compose | `:core:designsystem`、`:core:ui` 和 `:app` |
| 状态与异步 | ViewModel、Kotlin Coroutines、Flow、Turbine | ViewModel 和 Domain/UseCase；状态只通过 UiState 暴露 |
| 依赖注入 | Hilt、Hilt Compiler、Hilt Work | `:app` 与 Feature Module，使用构造函数注入 |
| 网络与序列化 | Retrofit、OkHttp、kotlinx.serialization、Logging Interceptor | `:core:network`；请求和 DTO 在 Feature Data |
| 数据存储 | Room、Room KSP、DataStore、Android Keystore | `:core:database`、`:core:datastore` |
| 分页与后台任务 | Paging 3、WorkManager | 对应 Feature Data |
| 图片与动画 | Coil Compose、Lottie Compose | `:core:ui` 和 Design System |
| 日志与诊断 | Timber、Sentry Android | `:core:analytics`；统一脱敏和采样 |
| 工程质量 | Detekt、Ktlint、Dependency Analysis、Gradle Version Catalog | 根 Gradle 构建和 CI |
| 测试 | JUnit 5、Coroutines Test、MockK、MockWebServer、Compose UI Test、Macrobenchmark | `test`、`androidTest` 和 `:benchmark` |

### 2.2 按功能启用的预置库

| 功能 | 库 | 固定要求 |
| --- | --- | --- |
| 音视频播放 | AndroidX Media3 | 仅存在媒体业务时启用，播放状态通过 Feature Repository 管理 |
| 相机 | CameraX | 仅存在相机业务时启用，权限和生命周期封装在平台适配层 |
| 推送 | Firebase Messaging | 仅对应 Google Play 渠道启用；多厂商推送通过独立 Adapter |
| 应用内购买 | Google Play Billing | 仅存在订阅业务时启用，权益校验由服务端完成 |

## 3. 标准目录结构

```text
Product/
├── app/                                      # Application 模块：唯一应用入口
│   ├── src/main/
│   │   ├── java/com/company/product/
│   │   │   ├── ProductApplication.kt         # @HiltAndroidApp，应用进程入口
│   │   │   ├── MainActivity.kt                # 根 Activity 和 Compose 宿主
│   │   │   ├── navigation/
│   │   │   │   ├── AppNavHost.kt              # 根 NavHost 和 Feature 路由注册
│   │   │   │   ├── AppRoute.kt                # 类型安全路由定义
│   │   │   │   └── DeepLinkHandler.kt         # Deep Link 转换与分发
│   │   │   ├── di/
│   │   │   │   └── AppModule.kt               # 全局依赖与 Feature Module 聚合
│   │   │   └── startup/
│   │   │       └── AppInitializer.kt          # 启动任务编排
│   │   └── res/                               # Manifest、主题、应用级资源
│   └── src/test/                              # Application 单元测试
├── feature/                                   # 按业务域拆分的 Feature 模块
│   ├── authentication/                        # 登录、注册、会话业务
│   ├── home/                                  # 首页业务
│   ├── project/                               # 项目业务
│   ├── user/                                  # 用户业务
│   └── settings/                              # 设置业务
├── core/
│   ├── common/                                # Result、Dispatcher、错误和通用扩展
│   │   └── src/main/kotlin/com/company/core/common/config/AppEnvironment.kt # 项目环境配置与第三方常量
│   ├── model/                                 # 跨 Feature 稳定模型和 ID 类型
│   ├── designsystem/                          # Material 3 主题、颜色、字号、间距和基础组件
│   ├── ui/                                    # 无业务语义的通用 Compose 组件
│   ├── network/                               # Retrofit、OkHttp、序列化和网络错误
│   ├── database/                              # Room Database、Migration 和通用数据库配置
│   ├── datastore/                             # DataStore、Keystore 和配置存储适配
│   ├── analytics/                              # 埋点、日志和性能监控适配
│   ├── testing/                               # Fake、Fixture、Dispatcher 和测试扩展
│   └── navigation/                            # 公共导航契约和导航扩展
├── build-logic/                               # Gradle Convention Plugin，统一模块构建配置
│   └── convention/
├── gradle/
│   ├── libs.versions.toml                     # 依赖版本唯一来源
│   └── wrapper/
├── benchmark/                                 # Macrobenchmark 和启动性能测试
├── e2e/                                       # 完整用户流程测试
├── config/                                    # Detekt、Ktlint、Lint 和基线配置
├── build.gradle.kts                           # 根项目构建配置
├── settings.gradle.kts                        # 模块注册和仓库配置
└── gradle.properties                          # Gradle 与 Kotlin 构建参数
```

## 4. Feature 模块内部结构

```text
feature/project/
├── build.gradle.kts                           # Feature 依赖和构建配置
└── src/
    ├── main/java/com/company/feature/project/
    │   ├── presentation/
    │   │   ├── ProjectRoute.kt                # 对外公开的路由入口
    │   │   ├── ProjectListScreen.kt           # Compose 页面
    │   │   ├── ProjectListViewModel.kt        # StateFlow、事件和用例调用
    │   │   ├── ProjectListUiState.kt          # 页面不可变状态
    │   │   ├── ProjectListUiEvent.kt          # 用户事件 sealed interface
    │   │   ├── ProjectListUiEffect.kt         # Snackbar、导航等一次性效果
    │   │   └── components/                    # 项目业务专属组件
    │   ├── domain/
    │   │   ├── model/Project.kt                # 领域实体
    │   │   ├── repository/ProjectRepository.kt # Repository 接口
    │   │   └── usecase/
    │   │       ├── GetProjectsUseCase.kt       # 查询项目用例
    │   │       └── CreateProjectUseCase.kt     # 创建项目用例
    │   ├── data/
    │   │   ├── remote/ProjectApi.kt            # Retrofit API
    │   │   ├── remote/ProjectDto.kt            # 网络 DTO
    │   │   ├── local/ProjectEntity.kt          # Room Entity
    │   │   ├── local/ProjectDao.kt              # Room DAO
    │   │   ├── mapper/ProjectMapper.kt         # DTO、Entity、Domain 转换
    │   │   └── repository/ProjectRepositoryImpl.kt # Repository 实现
    │   └── di/ProjectModule.kt                 # Hilt Module
    ├── test/                                   # Domain、UseCase、Mapper、ViewModel 单测
    └── androidTest/                            # Compose UI 和 Android 集成测试
```

## 5. 数据流和状态规则

```text
用户操作 → Compose Screen → ViewModel → UseCase → Repository
                                                        ↓
                                      Remote API / Room DAO / DataStore
结果 → Repository → UseCase → ViewModel StateFlow → Compose Screen
```

| 对象 | 固定规则 |
| --- | --- |
| Screen | 只收集 UiState、发送 UiEvent、渲染界面；禁止直接调用 Retrofit、Room 和 DataStore |
| ViewModel | 将事件转为 UseCase 调用，公开只读 StateFlow，使用 `stateIn` 管理状态生命周期 |
| UiState | 使用不可变 data class；用明确的加载、内容、空态和错误字段表达页面状态 |
| UiEvent | 使用 sealed interface 表达点击、刷新、重试、提交等用户意图 |
| UiEffect | 使用 SharedFlow 表达导航、Toast、Snackbar 等一次性效果 |
| UseCase | 一个业务意图对应一个 UseCase；只依赖 Domain Repository 接口 |
| Repository | 负责远端、本地、缓存、刷新和错误映射；向 Domain 返回业务模型 |
| DTO/Entity | 只存在 Data 层；必须通过 Mapper 转换，禁止传给 UI |
| Flow | 数据层使用 Flow 持续输出；ViewModel 在主线程安全地合并为 UiState |

## 6. 模块依赖规则

| 模块 | 允许依赖 | 禁止依赖 |
| --- | --- | --- |
| `:app` | 所有 Feature 和 Core | 被任何 Feature 或 Core 反向依赖 |
| `:feature:<name>` | `:core:*`；模块内部 presentation、domain、data | 其他 Feature；`app` |
| `presentation` | domain、core:ui、core:designsystem、core:common | data、Retrofit、Room、DataStore |
| `domain` | Kotlin、Coroutines、core:common、core:model | Android SDK、Compose、Retrofit、Room、Hilt |
| `data` | domain、core:network、core:database、core:datastore | presentation、app、其他 Feature |
| `:core:*` | 更基础的 Core 模块 | app、feature |

| 规则 | 固定要求 |
| --- | --- |
| Feature 公开入口 | 只暴露 Route、导航注册函数和必要的公共契约 |
| Feature 横向协作 | 由 app 编排；稳定共享类型进入 core:model |
| 依赖注入 | 所有生产类使用构造函数注入；Hilt Module 只负责绑定和创建 |
| Android Context | 仅在需要的位置注入，禁止在 Domain 保存 Context |
| 线程 | Repository 不阻塞主线程；IO 使用 `Dispatchers.IO`，计算使用专用 Dispatcher |
| 模块检查 | 使用 Gradle 依赖分析和 `api`/`implementation` 规则阻止反向依赖 |

## 7. 代码放置决策表

| 判断问题 | 唯一放置位置 |
| --- | --- |
| 是否为应用入口、根导航、Hilt 聚合或全局启动？ | `:app` |
| 是否属于认证、项目、订单、用户等明确业务域？ | `:feature:<name>` |
| 是否为该业务的页面、ViewModel、UiState、UiEvent 或业务组件？ | Feature 的 `presentation/` |
| 是否为该业务的实体、UseCase、Repository 接口或业务规则？ | Feature 的 `domain/` |
| 是否为该业务的 API、DTO、DAO、Entity、Mapper 或 Repository 实现？ | Feature 的 `data/` |
| 是否为无业务语义的网络、数据库、存储、UI 或测试能力？ | 对应 `:core:*` 模块 |
| 是否被两个及以上 Feature 稳定复用并含有业务语义？ | 提取到 `:core:model` 或独立业务能力模块 |
| 仍无法归属？ | 放在所属 Feature 的最小目录；禁止创建顶层 common、utils、managers、services |

## 8. 测试与质量检查

| 测试类型 | 固定位置 | 验证范围 |
| --- | --- | --- |
| Domain 单元测试 | `feature/<name>/src/test` | Entity、UseCase、业务规则和 Repository Fake |
| Data 测试 | `feature/<name>/src/test` | Mapper、Repository、缓存和错误映射 |
| ViewModel 测试 | `feature/<name>/src/test` | UiState、UiEvent、UiEffect 和协程取消 |
| Compose UI 测试 | `feature/<name>/src/androidTest` | 页面语义、交互、加载、空态和错误态 |
| 网络集成测试 | `:core:network/src/test` | MockWebServer、序列化和 HTTP 错误 |
| 数据库测试 | `:core:database/src/androidTest` | DAO、Migration、事务和索引 |
| 端到端测试 | `e2e/` 或独立测试模块 | 登录、核心业务流程和 Deep Link |
| CI 检查 | 根 Gradle 任务 | Lint、Detekt、Ktlint、单测、依赖边界和 APK 构建 |

## 9. 禁止的目录设计

| 禁止项 | 固定替代方式 |
| --- | --- |
| 顶层平铺 screens、viewmodels、repositories、services | 按业务放入 `:feature:<name>` |
| 在 Compose Screen 中直接请求网络或数据库 | 通过 ViewModel → UseCase → Repository |
| Domain 依赖 Android、Compose、Retrofit 或 Room | 将平台和技术实现留在 Data/Core |
| Feature 直接依赖 Feature | 由 `:app` 编排或提取稳定契约 |
| 建立无边界的全局 Store | 状态归属对应 Feature ViewModel |
| 使用 `api` 暴露全部实现依赖 | 默认使用 `implementation`，只公开必要接口 |
| 在 Repository 返回 DTO 或 Room Entity | 使用 Mapper 返回 Domain Model |
| 创建顶层 common、utils、managers 大杂物目录 | 按职责归入 Core 或 Feature |

## 10. 调研依据

| 来源 | 采用结论 |
| --- | --- |
| [Android Architecture Recommendations](https://developer.android.com/topic/architecture/recommendations) | 采用 UI 层、Data 层、Repository、ViewModel、StateFlow 和单向数据流。 |
| [Android Modularization](https://developer.android.com/topic/modularization/patterns) | 使用按功能拆分的 Feature 模块和按基础能力拆分的 Core 模块。 |
| [Compose Architecture](https://developer.android.com/develop/ui/compose/architecture) | Compose UI 通过事件上送、状态下发与 ViewModel 协作。 |
| [Guide to app architecture](https://developer.android.com/topic/architecture) | 中大型应用使用分层、可测试和依赖反转设计；Domain 层按复杂度建立。 |

生成新代码时，必须先使用代码放置决策表确定模块和层级，再确定具体目录。
