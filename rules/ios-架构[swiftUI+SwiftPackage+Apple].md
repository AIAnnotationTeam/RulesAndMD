# iOS 中大型项目架构规范

本模板面向新建的中大型 iOS 项目，固定采用 **SwiftUI + MVVM + 按业务域模块化 + Clean Architecture**。应用壳负责启动和跨业务编排，业务模块拥有展示、领域和数据实现，Swift Package Target 在编译期约束依赖方向。

本规范是确定的项目执行标准。Apple 官方提供 SwiftUI、Observation、Swift Concurrency 和 Swift Package 等机制；下面的分层与目录是基于这些机制及社区实践制定的团队约定，不代表 Apple 规定的唯一架构。

## 1. 固定技术配置

| 能力 | 固定配置 |
| --- | --- |
| 语言与系统基线 | Swift 6 语言模式，最低 iOS 17；开启严格并发检查 |
| 界面 | SwiftUI；平台专属控件由 `UIViewRepresentable`、`UIViewControllerRepresentable` 封装 |
| 页面模式 | MVVM；ViewModel 使用 `@MainActor @Observable` |
| 导航 | `NavigationStack` + 类型安全 Route + Coordinator |
| 并发 | `async/await`、结构化并发、`actor`、`AsyncSequence` |
| 网络与解析 | `URLSession` + `Codable` |
| 关系型持久化 | SwiftData；模型和 ModelContext 限定于数据实现层 |
| 轻量配置与敏感信息 | UserDefaults 保存非敏感偏好；Keychain 保存 Token 和凭据 |
| 依赖注入 | 构造函数注入；应用 Composition Root 显式组装对象 |
| 模块管理 | 本地 Swift Package + 独立 Target；第三方依赖通过 SPM 引入 |
| 测试 | Swift Testing 执行单元测试和集成测试；XCTest/XCUITest 执行 UI 自动化 |

## 2. 架构层级与职责

| 范围 | 位置 | 职责 |
| --- | --- | --- |
| Application | `App/` | 启动、依赖组装、根导航、Deep Link、跨 Feature 流程、系统生命周期 |
| Feature Presentation | `<Feature>Presentation` Target | SwiftUI View、ViewModel、不可变页面状态、页面交互、业务 UI |
| Feature Domain | `<Feature>Domain` Target | 领域模型、Use Case、Repository 协议、业务错误和规则 |
| Feature Data | `<Feature>Data` Target | API、DTO、持久化模型、Mapper、Repository 实现、缓存策略 |
| Feature Assembly | `<Feature>Assembly` Target | 组装本 Feature 的 Data、Domain、Presentation，对应用提供页面工厂 |
| Shared | `DesignSystem`、`SharedModels` Target | 无业务语义的复用 UI，以及有明确归属的跨域数据契约 |
| Infrastructure | `Networking`、`Persistence`、`SecureStorage`、`Observability` Target | 网络、存储接口、日志等无业务含义的技术能力 |

## 3. 标准项目目录

以下包目录按照 `Package.swift` 中 Target 的 `path` 显式配置；目录与 Target 一一对应，不能仅创建 Xcode Group 充当模块隔离。

```text
Product/                                  # 项目根目录，包含 Xcode 工程、源码包、测试和构建配置
├── Product.xcodeproj/                    # Xcode 工程文件，管理 App Target、签名和构建设置
├── App/                                  # iOS App Target；只负责启动、全局组装和跨业务编排
│   ├── ProductApp.swift                  # @main SwiftUI App，应用生命周期和根 View 入口
│   ├── Bootstrap/                        # 应用启动阶段的依赖组装和初始化
│   │   ├── AppContainer.swift            # 唯一全局 Composition Root，创建共享依赖实例
│   │   └── AppInitializer.swift          # 启动任务编排，例如配置、会话和监控初始化
│   ├── Navigation/                       # 根导航、跨业务跳转和外部链接处理
│   │   ├── AppCoordinator.swift          # 跨 Feature 导航与根页面切换，不实现业务规则
│   │   ├── AppRoute.swift                # 类型安全应用路由，只保存稳定页面参数
│   │   ├── AppRootView.swift             # NavigationStack 与目的地注册
│   │   └── DeepLinkHandler.swift         # URL、通知和外部事件到 AppRoute 的转换
│   ├── Lifecycle/                        # 系统生命周期和 UIApplicationDelegate 适配
│   │   └── AppDelegate.swift             # 推送注册等系统回调，通过适配器接入业务
│   ├── Session/                          # 登录态、用户切换和跨模块会话编排
│   │   └── SessionController.swift       # 会话状态、登出及跨模块数据清理编排
│   └── Resources/                        # App Target 专属资源
│       ├── Assets.xcassets               # App 图标、启动图和全局图片资源
│       ├── Localizable.xcstrings         # App 级本地化文案
│       └── PrivacyInfo.xcprivacy         # App 隐私清单声明
├── Packages/                             # 本地 Swift Package 容器
│   └── ProductModules/                   # 业务与基础能力的模块包
│       ├── Package.swift                 # Target、资源、平台版本和模块依赖声明
│       ├── Sources/                      # Swift Package 生产代码
│       │   ├── Features/                 # 按业务域划分的模块集合
│       │   │   ├── Project/              # 项目业务域，以下结构适用于每个 Feature
│       │   │   │   ├── ProjectDomain/    # 项目领域 Target：规则、模型和抽象协议
│       │   │   │   │   ├── Models/Project.swift                         # 领域实体，不包含 DTO 或 SwiftData 注解
│       │   │   │   │   ├── Repositories/ProjectRepository.swift         # Repository 协议，定义业务数据能力
│       │   │   │   │   ├── UseCases/GetProjectsUseCase.swift            # 查询项目列表的业务用例
│       │   │   │   │   ├── UseCases/CreateProjectUseCase.swift          # 创建项目的业务用例
│       │   │   │   │   └── Errors/ProjectError.swift                    # 项目领域错误和可处理业务异常
│       │   │   │   ├── ProjectData/      # 项目数据 Target：远程、本地和 Repository 实现
│       │   │   │   │   ├── Remote/ProjectAPI.swift                      # 项目接口定义和 URLSession 调用
│       │   │   │   │   ├── Remote/DTOs/ProjectDTO.swift                 # 网络响应 DTO，不直接暴露给 View
│       │   │   │   │   ├── Local/ProjectLocalStore.swift                # 项目本地数据读写适配器
│       │   │   │   │   ├── Local/Models/ProjectRecord.swift             # SwiftData 持久化模型
│       │   │   │   │   ├── Mappers/ProjectMapper.swift                  # DTO、Record 与 Domain Model 的转换
│       │   │   │   │   └── Repositories/DefaultProjectRepository.swift  # ProjectRepository 的具体实现
│       │   │   │   ├── ProjectPresentation/ # 项目展示 Target：View、ViewModel 和页面状态
│       │   │   │   │   ├── Screens/ProjectListView.swift                # 项目列表页面入口
│       │   │   │   │   ├── ViewModels/ProjectListViewModel.swift         # 页面状态、事件和异步任务管理
│       │   │   │   │   ├── State/ProjectListState.swift                 # 加载、成功、空态和错误等页面状态
│       │   │   │   │   ├── Components/ProjectCard.swift                 # 项目域专属 UI 组件
│       │   │   │   │   ├── Navigation/ProjectOutput.swift               # 页面向 App 输出的导航和事件契约
│       │   │   │   │   └── Resources/Localizable.xcstrings              # 项目 Feature 专属本地化文案
│       │   │   │   └── ProjectAssembly/   # 组装 Target，隐藏内部实现并输出页面工厂
│       │   │   │       └── ProjectFactory.swift                         # 创建注入完成的 View、ViewModel 和 Feature 入口
│       │   │   ├── Authentication/       # 认证业务域；使用相同的四层结构
│       │   │   ├── User/                 # 用户业务域；使用相同的四层结构
│       │   │   └── Settings/             # 设置业务域；使用相同的四层结构
│       │   ├── Shared/                   # 不含具体业务流程的跨 Feature 共享模块
│       │   │   ├── DesignSystem/         # Theme、设计 Token、Button、EmptyState 等基础 UI
│       │   │   └── SharedModels/         # UserID、TenantID 等跨业务稳定数据契约
│       │   └── Infrastructure/           # 无业务含义的技术基础设施模块
│       │       ├── Networking/           # URLSession、HTTPClient、Endpoint 和技术错误
│       │       ├── Persistence/          # 非业务存储协议、UserDefaults 和 SwiftData 基础适配
│       │       ├── SecureStorage/        # Keychain 封装，负责 Token 和凭据的安全读写
│       │       └── Observability/        # Logger、指标和监控 SDK 适配
│       └── Tests/                        # Swift Package 测试 Target
│           ├── ProjectDomainTests/       # 领域实体、规则和 Use Case 测试
│           ├── ProjectDataTests/         # Mapper、Repository、网络和持久化集成测试
│           ├── ProjectPresentationTests/ # ViewModel、页面状态和用户事件测试
│           └── NetworkingTests/          # HTTPClient、错误映射和请求行为测试
├── ProductUITests/                       # XCUITest App 流程测试，验证真实用户路径
├── Configurations/                       # 按构建环境拆分的 xcconfig 配置
│   ├── Debug.xcconfig                    # 开发环境 API、日志和调试开关
│   └── Release.xcconfig                  # 发布环境 API、签名和优化配置
├── Scripts/                              # 构建、代码生成、版本和 CI 辅助脚本
└── Docs/                                 # 架构决策、业务流程和开发约定文档
```

## 4. 编译依赖与公开入口

箭头仅表示源码导入方向。Repository 的运行时调用通过协议动态派发，Domain 不导入 Data。

```text
App → FeatureAssembly → FeaturePresentation → FeatureDomain
                    └→ FeatureData ─────────→ FeatureDomain
FeatureData → Infrastructure
FeaturePresentation → DesignSystem
FeatureDomain → SharedModels
```

| 模块 | 允许依赖 | 禁止依赖 |
| --- | --- | --- |
| App | Feature Assembly、Shared、Infrastructure | Feature Data 和 Presentation 内部实现 |
| Feature Assembly | 本 Feature 的 Presentation、Domain、Data 与技术模块 | 其他 Feature 的实现 |
| Feature Presentation | 本 Feature Domain、DesignSystem | Data、URLSession、SwiftData、Keychain、App |
| Feature Domain | Swift 标准库、Foundation 的平台无关类型、SharedModels | SwiftUI、UIKit、Observation、SwiftData、网络及存储 SDK |
| Feature Data | 本 Feature Domain、Infrastructure | Presentation、App、其他 Feature Data |
| Shared | 更基础的 Shared Target | Feature、App、具体业务 Repository |
| Infrastructure | 更基础的 Infrastructure Target | Feature、Shared UI、App |

| 约束 | 执行标准 |
| --- | --- |
| Feature 横向依赖 | 禁止 Feature 依赖其他 Feature；跨域流程由 App Coordinator 编排，通过 ID、输入输出契约传递信息。 |
| 对外入口 | App 通过 `ProjectFactory` 获取页面；跨域跳转通过公开的 `ProjectOutput` 回调交给 App。 |
| 可见性 | 声明默认保持 `internal`；仅跨 Target 所需的协议、模型、工厂与初始化器声明为 `public`。 |
| 依赖注入 | AppContainer 创建技术客户端；Feature Factory 注入 Repository 实现、Use Case 和 ViewModel。禁止全局 Service Locator。 |
| 第三方 SDK | 只在负责该能力的 Target 中声明与包装；Domain 和 ViewModel 不暴露 SDK 类型。 |
| 共享契约 | 跨域且含业务含义的稳定 ID 和值对象放 SharedModels；仅属于一个业务的模型留在其 Domain。 |

## 5. 数据流、状态与并发

```text
用户操作 → View → ViewModel → Use Case → Repository 协议
                                          ↓ 实现注入
                                  DefaultRepository
                                   ↓             ↓
                               HTTPClient     LocalStore
结果 → Domain Model → ViewModel 更新 State → View 刷新
```

| 对象 | 固定规则 |
| --- | --- |
| View | 渲染状态、收集输入、触发 ViewModel 方法；页面拥有的 ViewModel 用 `@State` 持有；不在 `body` 发起请求。 |
| ViewModel | 使用 `@MainActor @Observable`；公开只读页面状态和事件方法，负责加载、重试、分页与展示错误转换。 |
| 页面状态 | 在 Feature Presentation 的 `State/` 定义值类型；包含加载、数据、错误和空态，禁止以多个互相矛盾的布尔值表示请求阶段。 |
| 表单局部状态 | 仅用于焦点、展开、临时输入的值由 View 的 `@State` 持有；提交验证由 ViewModel 调用领域规则。 |
| Use Case | 位于 Domain，每个类型对应一个业务意图；通过构造函数获得 Repository 协议，使用 `async throws` 输出领域结果。 |
| Repository | Domain 定义协议，Data 提供实现；协调远端与缓存，返回领域对象，不将 DTO 和 SwiftData 模型传给展示层。 |
| 共享可变数据 | 使用 actor 隔离；SwiftData 后台读写通过 `@ModelActor`，不跨隔离域传递 ModelContext 和持久化对象。 |
| 网络与计算 | 异步网络使用 URLSession；同步解码和重计算在数据层的非主隔离执行环境运行，不能仅凭 `async` 假定已离开主线程。 |
| 任务生命周期 | 页面请求由 `.task` 启动并传递取消；ViewModel 自持任务必须取消前一次同类请求，并防止过期响应覆盖新状态。 |
| Swift 6 并发 | 跨隔离域传递的值满足 `Sendable`；禁止使用 `@unchecked Sendable` 掩盖未同步的共享状态。 |
| 错误与日志 | Data 将传输错误映射为领域可处理结果；ViewModel 转换为用户提示；日志隐藏 Token、凭据和个人敏感信息。 |

## 6. 导航、平台能力与持久化

| 场景 | 固定归属 | 规则 |
| --- | --- | --- |
| 根路由和 Deep Link | `App/Navigation` | Route 使用 `Hashable` 值类型，参数只传稳定 ID，不传 ViewModel、DTO 或数据库对象。 |
| Feature 内流程 | Feature Presentation 的 `Navigation/` | 管理域内页面状态；通过输出回调通知 App 跨 Feature 跳转。 |
| 登录与退出 | `App/Session` | App 编排认证 Feature、Keychain 更新、会话状态与缓存清理；服务端仍负责鉴权。 |
| 摄像头、相册、定位、蓝牙 | 按能力建立 Infrastructure Target | 定义最小业务需要的协议，平台 SDK 的权限、线程和错误在适配器中处理。 |
| UIKit 互操作 | 使用方 Presentation 的 `Components/Platform/` | 业务专属包装留在 Feature；跨业务无语义包装进入 DesignSystem。 |
| 业务数据库 | 所属 Feature Data 的 `Local/` | Feature 拥有自己的 SwiftData Schema、ModelContainer 和 Migration；不将业务模型下沉至通用 Persistence。 |
| 事务边界 | 同一 Feature Data | 需要原子事务的实体必须在同一业务域和 ModelContainer；跨域更新通过显式工作流与补偿处理。 |
| 用户偏好 | Infrastructure Persistence | UserDefaults 仅保存非敏感设置，业务 Key 由所属 Feature 定义。 |
| 图片和文件 | 所属 Feature Data | 下载、缓存失效和生命周期归数据层；UI 仅接收展示需要的资源标识。 |
| 资源与本地化 | 所属 Target 的 `Resources/` | SPM 使用 `.process` 声明资源并经 `Bundle.module` 访问；应用图标归 App。 |

## 7. 新代码归属决策表

先根据职责选中具体行；“属于某业务”只确定 Feature，随后必须继续确定其内部 Target，不能停留在 Feature 根目录。

| 代码职责 | 唯一归属 |
| --- | --- |
| 启动、系统生命周期、跨域导航、Deep Link、全局组装 | `App/` 对应目录 |
| 业务页面、业务组件、ViewModel、展示状态 | `<Feature>Presentation` |
| 实体、业务校验、Use Case、Repository 协议 | `<Feature>Domain` |
| 接口定义、DTO、SwiftData 模型、Mapper、Repository 实现 | `<Feature>Data` |
| Feature 内对象创建和对 App 暴露的页面工厂 | `<Feature>Assembly` |
| 无业务含义且跨 Feature 复用的 UI、设计 Token | `DesignSystem` |
| 跨域稳定的数据契约与 ID | `SharedModels` |
| HTTP、通用存储、Keychain、日志、系统能力适配 | 对应 Infrastructure Target |
| 单元与集成测试 | `Tests/<被测Target>Tests/` |
| 完整 App 用户流程测试 | `ProductUITests/` |

禁止创建顶层 `Managers`、`Helpers`、`Services`、`ViewModels` 大集合。无法直接分类时，先拆分混合职责，再按照上表分别放置；不得使用“临时公共目录”绕过归属。

## 8. 自动化检查与测试

| 检查 | 执行标准 |
| --- | --- |
| 编译边界 | `Package.swift` 显式声明 Target 依赖；CI 校验 Feature 无横向依赖、Domain 无 SDK 实现依赖。 |
| 单元测试 | Swift Testing 验证领域规则、Use Case、Mapper 和 ViewModel；使用 Repository 协议的 Fake，不进行真实网络调用。 |
| 数据集成测试 | 使用内存 SwiftData 容器和 URLProtocol Stub，验证映射、持久化、迁移与 HTTP 错误处理。 |
| 导航测试 | 验证 Route 解析、Feature Output、登录跳转和非法 Deep Link。 |
| UI 自动化 | XCUITest 验证登录、关键创建流程、错误重试；使用独立测试启动参数注入测试环境。 |
| CI 构建 | 在 iOS Simulator 上构建 App 与包 Target，运行 Swift Testing 和 XCUITest；SwiftUI 包不以 macOS 下裸 `swift test` 代替 iOS 验证。 |

## 9. 联网调研依据与采用结论

| 来源 | 核实内容与本模板采用的结论 |
| --- | --- |
| [Apple：Managing model data in your app](https://developer.apple.com/documentation/swiftui/managing-model-data-in-your-app) | 数据模型与 View 分离有利于模块化和测试；使用 Observation 跟踪模型变化，本模板据此固定使用 Observable ViewModel。 |
| [Apple：NavigationStack](https://developer.apple.com/documentation/swiftui/navigationstack) | 提供栈式 SwiftUI 导航；本模板以类型安全 Route 和 Coordinator 组织导航。 |
| [Apple：Swift Testing](https://developer.apple.com/documentation/testing) | 支持 Swift Package 与 Xcode 项目测试；作为本模板单元与集成测试框架。 |
| [社区：iOS Clean Architecture + MVVM](https://github.com/kudoleh/iOS-Clean-Architecture-MVVM) | 公开实践采用 Presentation、Domain、Data，Repository 协议归 Domain、实现归 Data，结合构造注入和 Coordinator。模板采纳这些边界，不沿用其旧 UI 和绑定实现。 |

调研结论：Apple 没有规定统一的应用分层。按业务域模块化、MVVM、Repository 与依赖反转是能够组合使用的主流实践；本模板将这些实践固化为 SPM Target 边界及明确的代码归属规则。
