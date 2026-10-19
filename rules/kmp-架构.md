# Kotlin Multiplatform 中大型项目架构规范

本规范适用于 Android、iOS 与 Desktop 三端的 Kotlin Multiplatform 中大型项目。项目固定使用 Compose Multiplatform 共享 UI、Navigation Compose Multiplatform 管理导航、Koin 管理依赖注入、Ktor Client 访问网络、Room KMP 管理本地数据库、StateFlow 驱动界面状态，并使用 Gradle 多模块组织源码。Desktop 目标固定使用 JVM Desktop、Ktor CIO 与 Room Bundled SQLite Driver。

采用**按业务域模块化 + Feature 内 MVVM 与 Clean Architecture**。业务模块是代码归属的第一边界；UI、用例、领域模型与数据实现是模块内部的第二边界。平台差异只在 Kotlin Source Set 边界出现，不向业务代码扩散。

## 1. 架构层级

| 层级 | Gradle 模块 | 职责 | 允许依赖 | 禁止依赖 |
| --- | --- | --- | --- | --- |
| Application | `:androidApp`、`:iosApp`、`:desktopApp`、`:composeApp` | 应用启动、根 Compose UI、导航图、Koin 启动、平台入口 | `feature:*`、`core:*` | 被 Feature 或 Core 反向依赖 |
| Feature | `:feature:<feature>` | 完整业务域：页面、ViewModel、用例、领域规则、Repository 实现 | 自身、`core:*` | 其他 Feature 内部实现、Application |
| Core | `:core:*` | 跨 Feature 复用的 UI、网络、数据库、模型、平台适配、设计系统 | 更基础的 Core 模块 | Application、Feature |
| Platform | 每个模块的 `androidMain`、`iosMain`、`desktopMain` | Android、iOS 与 Desktop 专属 API、actual 实现、平台 SDK 初始化 | 当前模块的 `commonMain` 与平台依赖 | 其他 Feature 的内部实现 |

| Feature 内层 | 目录 | 职责 |
| --- | --- | --- |
| Presentation | `presentation` | Compose Screen、Feature 专属组件、ViewModel、UiState、UiEvent、UiEffect |
| Domain | `domain` | 实体、值对象、Repository 接口、Use Case、纯业务规则 |
| Data | `data` | Ktor API、Room DAO、DTO、Entity、Mapper、Repository 实现 |
| DI | `di` | Feature 的 Koin Module；组装本 Feature 内部依赖 |

## 2. 根目录与模块结构

```text
project/
├── androidApp/                                  # Android 启动工程与 Manifest
│   └── src/main/kotlin/.../MainActivity.kt
├── iosApp/                                      # iOS Xcode 工程与 Swift 入口
│   └── iosApp/
│       └── iOSApp.swift
├── desktopApp/                                  # Desktop JVM 启动与打包模块
│   └── src/main/kotlin/.../Main.kt
├── composeApp/                                  # 应用编排模块
│   └── src/
│       ├── commonMain/kotlin/com/company/app/
│       │   ├── App.kt                           # 根 Compose UI
│       │   ├── di/AppModule.kt                  # 聚合 Koin Module
│       │   ├── navigation/
│       │   │   ├── AppNavHost.kt                # 根导航图
│       │   │   ├── Routes.kt                    # 类型安全路由定义
│       │   │   └── NavExtensions.kt
│       │   └── startup/
│       │       └── AppInitializer.kt
│       ├── androidMain/kotlin/com/company/app/  # 共享 App 模块的 Android actual 实现
│       ├── iosMain/kotlin/com/company/app/      # 共享 App 模块的 iOS actual 实现
│       ├── desktopMain/kotlin/com/company/app/  # 共享 App 模块的 Desktop actual 实现
│       └── commonTest/kotlin/com/company/app/
├── feature/                                     # 按业务域拆分的 KMP 模块
│   ├── authentication/
│   ├── project/
│   ├── user/
│   ├── notification/
│   └── settings/
├── core/                                        # 跨业务域基础模块
│   ├── common/                                  # Result、Dispatcher、基础类型与工具
│   ├── model/                                   # 跨 Feature 稳定共享的领域模型
│   ├── designsystem/                            # Theme、Token、通用 Compose 组件
│   ├── navigation/                              # 导航抽象与公共导航组件
│   ├── network/                                 # Ktor Client、认证拦截、错误映射
│   ├── database/                                # Room Database、Driver、Migration
│   ├── datastore/                               # Settings、KeyValue 存储适配
│   ├── platform/                                # 平台能力抽象与 expect 声明
│   └── testing/                                 # 测试 Fixture、Fake、Coroutine Rule
├── build-logic/                                 # 统一 Gradle Convention Plugin
│   └── convention/
├── gradle/
│   └── libs.versions.toml                       # 唯一版本目录
├── e2e/                                         # Maestro 端到端测试
├── docs/
├── settings.gradle.kts
├── build.gradle.kts
└── gradle.properties
```

## 3. Feature 标准目录模板

每个 Feature 都是独立 KMP Gradle 模块，目录一致。没有业务归属的代码不得进入 Feature。

```text
feature/project/
├── build.gradle.kts
└── src/
    ├── commonMain/
    │   ├── kotlin/com/company/feature/project/
    │   │   ├── presentation/
    │   │   │   ├── screen/
    │   │   │   │   ├── ProjectListScreen.kt
    │   │   │   │   └── ProjectDetailScreen.kt
    │   │   │   ├── component/
    │   │   │   │   └── ProjectCard.kt
    │   │   │   ├── ProjectListViewModel.kt
    │   │   │   ├── ProjectListUiState.kt
    │   │   │   ├── ProjectListUiEvent.kt
    │   │   │   └── ProjectListUiEffect.kt
    │   │   ├── domain/
    │   │   │   ├── model/
    │   │   │   │   └── Project.kt
    │   │   │   ├── repository/
    │   │   │   │   └── ProjectRepository.kt
    │   │   │   └── usecase/
    │   │   │       ├── GetProjectsUseCase.kt
    │   │   │       └── CreateProjectUseCase.kt
    │   │   ├── data/
    │   │   │   ├── remote/
    │   │   │   │   ├── ProjectApi.kt
    │   │   │   │   └── dto/ProjectDto.kt
    │   │   │   ├── local/
    │   │   │   │   ├── ProjectDao.kt
    │   │   │   │   └── entity/ProjectEntity.kt
    │   │   │   ├── mapper/
    │   │   │   │   └── ProjectMapper.kt
    │   │   │   └── repository/
    │   │   │       └── ProjectRepositoryImpl.kt
    │   │   ├── di/
    │   │   │   └── ProjectModule.kt
    │   │   └── ProjectFeature.kt                # Feature 的唯一公开入口
    │   └── resources/
    ├── androidMain/
    │   └── kotlin/com/company/feature/project/
    │       └── ProjectPlatformModule.kt
    ├── iosMain/
    │   └── kotlin/com/company/feature/project/
    │       └── ProjectPlatformModule.kt
    ├── desktopMain/
    │   └── kotlin/com/company/feature/project/
    │       └── ProjectPlatformModule.kt
    ├── commonTest/
    │   └── kotlin/com/company/feature/project/
    ├── androidUnitTest/
    ├── iosTest/
    └── desktopTest/
```

## 4. Kotlin Source Set 规则

| Source Set | 放置内容 | 禁止内容 |
| --- | --- | --- |
| `commonMain` | 全端共享的 Compose UI、ViewModel、Use Case、Repository 接口与实现、Ktor 调用、Room DAO、Koin Module | `android.*`、`java.*`、UIKit、Swift、Android Context 等平台 API |
| `androidMain` | Android Context、Android Room Driver、OkHttp Engine、Android 权限与系统能力的 actual 实现 | 业务规则、Feature Screen、可共享的 Use Case |
| `iosMain` | iOS Room Driver、Darwin Engine、iOS 系统能力的 actual 实现、Swift/Objective-C 互操作入口 | 业务规则、Feature Screen、可共享的 Use Case |
| `desktopMain` | JVM Desktop Window、Room Bundled SQLite Driver、CIO Engine、桌面文件与系统能力的 actual 实现 | 业务规则、Feature Screen、可共享的 Use Case |
| `commonTest` | Domain、Use Case、Mapper、ViewModel 的跨端单元测试 | Android、iOS、Desktop 平台测试 API |
| `androidUnitTest` | Android 专属平台适配测试 | 跨端业务规则测试 |
| `iosTest` | iOS 专属平台适配测试 | 跨端业务规则测试 |
| `desktopTest` | Desktop 专属平台适配测试 | 跨端业务规则测试 |

| 平台能力场景 | 固定实现 |
| --- | --- |
| 共享层声明 | 在 `core:platform/commonMain` 使用 `expect` 声明接口或工厂。 |
| Android 实现 | 在对应模块的 `androidMain` 使用 `actual` 实现，并封装 Android SDK。 |
| iOS 实现 | 在对应模块的 `iosMain` 使用 `actual` 实现，并封装 iOS SDK。 |
| Desktop 实现 | 在对应模块的 `desktopMain` 使用 `actual` 实现，并封装 JVM Desktop API。 |
| 业务层调用 | 只依赖 `commonMain` 声明，禁止检查平台类型并分支调用。 |

## 5. 数据与状态流

```text
Compose Screen → ViewModel → Use Case → Repository Interface
                                      ↓
                              Repository Implementation
                                      ↓
                          Ktor API / Room DAO / DataStore
```

| 模块 | 固定职责 | 禁止事项 |
| --- | --- | --- |
| Compose Screen | 收集 `StateFlow<UiState>`，渲染界面，发送 `UiEvent` | 直接调用 Ktor、Room、Repository 或 Use Case 以外的数据层对象 |
| ViewModel | 将 `UiEvent` 转为 Use Case 调用，维护 `UiState`，通过 `UiEffect` 发出一次性效果 | 存放 DTO、SQL、导航图定义 |
| Use Case | 一个用户意图对应一个业务动作，执行领域规则 | 调用 Compose API、访问平台 API、返回 DTO |
| Repository Interface | 在 Domain 定义业务需要的数据契约 | 暴露 Ktor、Room、DTO、Entity 类型 |
| Repository Implementation | 在 Data 实现远端与本地数据协调、缓存和 Mapper | 被其他 Feature 的内部代码直接导入 |
| Ktor API / Room DAO | 执行协议请求与数据库读写 | 包含业务决策与界面状态 |

| 状态类型 | 固定位置 | 固定实现 |
| --- | --- | --- |
| Screen 状态 | `presentation/<Feature>UiState.kt` | `StateFlow<UiState>`，由 ViewModel 单向输出。 |
| 用户事件 | `presentation/<Feature>UiEvent.kt` | sealed interface，由 Screen 发送给 ViewModel。 |
| 一次性效果 | `presentation/<Feature>UiEffect.kt` | `SharedFlow<UiEffect>`，承载导航、Toast、系统权限请求。 |
| 列表刷新与加载 | ViewModel 的 `UiState` | 使用不可变状态对象，不建立全局页面状态 Store。 |
| 跨 Feature 会话状态 | `core:common` 的 `SessionManager` | 使用 `StateFlow` 暴露只读会话状态。 |
| 本地设置与 Token | `core:datastore` | 设置数据使用 DataStore；Token 使用平台安全存储实际实现。 |

## 6. 导航与依赖注入

### 6.1 Navigation Compose Multiplatform

| 内容 | 固定位置 | 规则 |
| --- | --- | --- |
| 类型安全路由 | `composeApp/.../navigation/Routes.kt` | 每个目的地使用 `@Serializable` 对象或数据类定义。 |
| 根导航图 | `composeApp/.../navigation/AppNavHost.kt` | 只注册 Feature 公开入口和嵌套路由图。 |
| Feature 导航入口 | `feature/<feature>/.../ProjectFeature.kt` | 对外导出 Screen 与导航注册函数，不暴露内部 Screen 文件路径。 |
| 导航触发 | ViewModel 发送 `UiEffect.Navigate`，Screen 调用 `NavController` | ViewModel 不持有 `NavController`。 |
| Deep Link | `composeApp/.../navigation/AppNavHost.kt` | 与目的地一起声明，并使用端到端测试覆盖。 |

### 6.2 Koin

| 内容 | 固定位置 | 规则 |
| --- | --- | --- |
| Core Module | 每个 `core:*` 模块的 `commonMain/.../di` | 注册该模块对外提供的单例与工厂。 |
| Feature Module | `feature/<feature>/commonMain/.../di/<Feature>Module.kt` | 注册 Repository 实现、Use Case、ViewModel。 |
| 应用聚合 | `composeApp/.../di/AppModule.kt` | 唯一聚合全部 Core 与 Feature Module 的位置。 |
| Android 启动 | `androidApp/MainActivity.kt` | 在创建 Compose 内容前启动 Koin。 |
| iOS 启动 | `iosApp/iOSApp.swift` | 在创建 Compose `UIViewController` 前调用共享层 Koin 启动入口。 |
| Desktop 启动 | `desktopApp/src/main/kotlin/.../Main.kt` | 在创建 Compose Window 前启动 Koin。 |
| 注入规则 | 所有业务类使用构造函数注入 | 禁止在 Domain、Use Case、Repository 中使用 `KoinComponent` 定位依赖。 |

## 7. Core 模块职责

| 模块 | 职责 | 禁止放入 |
| --- | --- | --- |
| `core:common` | `AppResult`、Coroutine Dispatcher、时钟、基础错误、SessionManager | Feature 专属模型、业务用例 |
| `core:model` | 两个及以上 Feature 稳定共享的领域模型 | API DTO、Room Entity、仅单 Feature 模型 |
| `core:designsystem` | Theme、颜色、间距、字体、无业务语义 Compose 组件 | `ProjectCard`、`UserRoleBadge` 等业务组件 |
| `core:navigation` | 通用导航契约和公共导航 UI | Feature 路由图、业务导航逻辑 |
| `core:network` | Ktor Client、认证插件、序列化、网络错误映射 | Feature API、业务 DTO、业务请求参数 |
| `core:database` | Room Database、Driver、Migration、通用 DAO 基类 | Feature 专属 DAO、Entity、查询语义 |
| `core:datastore` | 设置存储、安全存储接口与平台 actual 实现 | Feature 专属缓存键与业务缓存策略 |
| `core:platform` | 平台能力的 expect 声明和 actual 实现 | 业务规则、Feature UI |
| `core:testing` | Fixture、Fake、Test Dispatcher、Compose 测试工具 | 生产业务代码 |

## 8. 代码放置决策表

新增文件按顺序判断，命中第一项即确定位置。

| 顺序 | 判断问题 | 放置位置 |
| --- | --- | --- |
| 1 | 是否为 Android、iOS 或 Desktop 应用启动、Manifest、Xcode 配置、桌面打包配置？ | 对应 `androidApp/`、`iosApp/`、`desktopApp/` |
| 2 | 是否为根 UI、根导航图、Koin 聚合或应用启动编排？ | `composeApp/` |
| 3 | 是否属于认证、用户、项目、订单、账单等一个明确业务域？ | `feature/<feature>/` |
| 4 | 是否为该业务的 Screen、组件、ViewModel、UiState、UiEvent、UiEffect？ | `feature/<feature>/presentation/` |
| 5 | 是否为该业务的实体、Repository 接口、Use Case 或纯规则？ | `feature/<feature>/domain/` |
| 6 | 是否为该业务的 API、DTO、DAO、Entity、Mapper 或 Repository 实现？ | `feature/<feature>/data/` |
| 7 | 是否为两个及以上 Feature 稳定复用且没有业务语义的能力？ | 对应 `core:*` 模块 |
| 8 | 是否直接访问 Android、iOS、Desktop 或平台 SDK？ | 当前模块的 `androidMain/`、`iosMain/`、`desktopMain/` |
| 9 | 是否无法归属？ | 放在最接近的 Feature 内部；禁止新建顶层 `common`、`utils`、`service`、`manager` 杂物目录。 |

## 9. 模块依赖规则

```text
androidApp / iosApp / desktopApp → composeApp → feature:* → core:*
                                                └──────────→ core:*
core:* → 更基础的 core:*
```

| 规则 | 要求 |
| --- | --- |
| Feature 间依赖 | 禁止 Feature 直接依赖另一个 Feature；跨业务编排只在 `composeApp` 完成。 |
| Feature 公开 API | `ProjectFeature.kt` 是 Feature 唯一跨模块入口；禁止跨模块导入 `presentation`、`domain`、`data` 的内部文件。 |
| Domain 依赖 | Domain 只能依赖 Kotlin 标准库、Coroutines 和 `core:common`、`core:model`。 |
| Data 依赖 | Data 可以依赖 Domain、`core:network`、`core:database`、`core:datastore`。 |
| Presentation 依赖 | Presentation 可以依赖 Domain、`core:designsystem`、`core:common`；不得直接依赖 Ktor、Room、DataStore。 |
| Core 依赖 | Core 禁止依赖 `composeApp` 与 Feature。 |
| 自动检查 | 使用 `dependency-analysis-gradle-plugin` 与 Gradle 模块依赖检查阻止反向依赖和未使用依赖。 |


## 11. 测试边界

| 测试类型 | 固定位置 | 验证范围 |
| --- | --- | --- |
| 跨端单元测试 | `<module>/src/commonTest/` | Use Case、Domain 规则、Mapper、ViewModel 状态转换 |
| Android 平台测试 | `<module>/src/androidUnitTest/` | Android actual、Room Android Driver、Android 平台适配 |
| iOS 平台测试 | `<module>/src/iosTest/` | iOS actual、Room iOS Driver、iOS 平台适配 |
| Desktop 平台测试 | `<module>/src/desktopTest/` | Desktop actual、Room Bundled SQLite Driver、Desktop 平台适配 |
| Compose UI 测试 | Feature 的 `commonTest` | Screen 的加载、错误、空态和用户事件 |
| 端到端测试 | `e2e/` | 登录、关键创建流程、支付和 Deep Link |

