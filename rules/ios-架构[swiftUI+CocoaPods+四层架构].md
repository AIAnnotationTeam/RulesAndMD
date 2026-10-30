# iOS SwiftUI + CocoaPods 四层架构项目规范

## 第三方配置管理（强制）

第三方服务地址、应用 ID、API Key 等项目配置统一定义在 `Infrastructure/Configuration/AppEnvironment.swift` 的 `AppEnvironment` 中，其他代码统一引用，禁止分散定义；本工程不强制使用环境文件。

本模板面向 SwiftUI + CocoaPods 的中大型 iOS 项目，固定采用传统四层架构。

| 层级 | 名称 | 目录 | 核心职责 |
| --- | --- | --- | --- |
| L0 | Application | `App/` | 应用启动、全局组装、根导航、跨业务编排 |
| L1 | Business | `Business/` | 具体业务模块、页面、交互和业务流程 |
| L2 | Business Architecture | `BusinessArchitecture/` | 跨业务复用的标准化业务能力 |
| L3 | Infrastructure | `Infrastructure/` | 网络、存储、日志、系统能力和 CocoaPods SDK 适配 |

依赖方向固定为：`L0 → L1 → L2 → L3`。L0 可以直接依赖 L2、L3；L1 可以直接依赖 L3。L3 禁止依赖 L2、L1、L0；L2 禁止依赖 L1、L0；L1 禁止依赖 L0。L1 和 L2 模块禁止横向强依赖。

## 1. 固定技术配置

| 能力 | 固定配置 |
| --- | --- |
| UI | SwiftUI |
| 页面模式 | MVVM；ViewModel 使用 `@MainActor @Observable` |
| 导航 | `NavigationStack` + 类型安全 Route + Application Coordinator |
| 并发 | `async/await`、结构化并发、actor、AsyncSequence |
| 网络与解析 | URLSession + Codable |
| 数据持久化 | SwiftData；业务模型放 L1，底层适配放 L3 |
| 配置存储 | UserDefaults 保存非敏感偏好 |
| 敏感信息 | Keychain |
| 第三方依赖 | CocoaPods，统一声明在根目录 Podfile |
| 依赖注入 | L0 显式组装，业务模块使用构造函数注入 |
| 测试 | Swift Testing 或 XCTest；XCUITest 执行 UI 自动化 |

## 1.1 默认第三方库基线

| 能力 | CocoaPods 默认库 | 固定归属 |
| --- | --- | --- |
| 网络请求 | `Alamofire` | L3 `Infrastructure/Networking`，统一封装请求、重试和错误 |
| 图片加载 | `Kingfisher` | L3 Adapter，L1 只使用图片资源接口 |
| 动画 | `lottie-ios` | L3 Adapter，业务层只传动画资源和播放状态 |
| 崩溃与性能 | `Firebase/Crashlytics`、`Firebase/Performance` | L3 `Infrastructure/Observability`，禁止在业务层直接调用 Firebase |
| 代码质量 | `SwiftLint` | CI 和 Xcode Build Phase，统一 Swift 代码检查 |

### 1.2 按功能启用的预置库

| 功能 | CocoaPods 库 | 固定要求 |
| --- | --- | --- |
| WebSocket | `Starscream` | 只有系统 `URLSessionWebSocketTask` 不满足协议要求时启用，统一放 L3 Adapter |
| 订阅支付 | `RevenueCat` | 仅存在订阅业务时启用，购买和权益校验通过 L3 服务封装 |
| 富文本/Markdown | `Down` | 仅对应内容功能启用，解析结果不得直接泄漏到 Domain |
| 音频/蓝牙 | 项目确认的音频或蓝牙 Pod | 通过 L3 协议封装，权限、连接和资源释放不进入 L1 |

## 2. 四层职责

| 层级 | 放置内容 | 允许依赖 | 禁止内容 |
| --- | --- | --- | --- |
| L0 Application | App 入口、AppDelegate、根导航、Deep Link、全局依赖组装 | L1、L2、L3 | 具体业务规则和业务查询 |
| L1 Business | 业务 View、ViewModel、State、Service、Validator、业务 Model | L2、L3 | L0、其他 L1 内部实现 |
| L2 Business Architecture | Account、Session、Organization、Permission、标准业务组件 | L3 | L0、L1、具体业务字段和流程 |
| L3 Infrastructure | HTTPClient、SwiftData 底层适配、Keychain、日志、文件、Pod Adapter | 系统 SDK、CocoaPods、L3 | L0、L1、L2 和业务判断 |

## 3. 标准目录结构

```text
Product/                                      # 项目根目录
├── Product.xcworkspace/                      # CocoaPods 生成的 Workspace，必须用它打开项目
├── Product.xcodeproj/                        # Xcode 工程配置
├── Podfile                                   # CocoaPods 依赖声明
├── Podfile.lock                              # CocoaPods 锁定版本，必须提交
├── Pods/                                     # CocoaPods 生成目录，禁止直接修改
├── App/                                      # L0 Application：入口和全局编排
│   ├── ProductApp.swift                      # @main SwiftUI 应用入口
│   ├── AppDelegate.swift                     # 推送和系统生命周期适配
│   ├── Bootstrap/                            # 启动初始化和依赖组装
│   │   ├── AppContainer.swift                # 唯一 Composition Root
│   │   └── AppInitializer.swift              # 配置、会话、监控等启动任务
│   ├── Navigation/                           # 根导航和跨业务跳转
│   │   ├── AppCoordinator.swift              # 跨业务流程编排
│   │   ├── AppRoute.swift                    # 类型安全路由
│   │   ├── AppRootView.swift                 # NavigationStack 根视图
│   │   └── DeepLinkHandler.swift             # 外部链接转换为 AppRoute
│   ├── Session/                              # 全局会话编排
│   │   └── SessionController.swift           # 登录、登出和缓存清理
│   └── Resources/                            # App 图片、权限和本地化资源
│       ├── Assets.xcassets                   # App 图标和全局图片
│       ├── Localizable.xcstrings             # App 级本地化文案
│       └── PrivacyInfo.xcprivacy             # 隐私清单
├── Business/                                 # L1 Business：具体业务模块
│   ├── Authentication/                       # 登录、注册和找回密码
│   │   ├── Views/                            # 认证页面和业务组件
│   │   ├── ViewModels/                       # 页面状态和交互逻辑
│   │   ├── Models/                           # 认证业务模型
│   │   ├── Services/                         # 认证业务流程
│   │   ├── Validators/                       # 认证输入校验
│   │   └── AuthenticationEntry.swift         # 对 L0 暴露的业务入口
│   ├── Project/                              # 项目业务，使用相同目录结构
│   ├── User/                                 # 用户业务，使用相同目录结构
│   └── Settings/                             # 设置业务，使用相同目录结构
├── BusinessArchitecture/                     # L2：跨业务标准能力
│   ├── Account/                              # 账户信息和账户状态
│   ├── Authentication/                       # 标准认证、会话和权限
│   ├── Organization/                         # 组织、租户和成员上下文
│   ├── File/                                 # 业务级文件上传和下载
│   ├── Components/                           # 跨业务标准 UI 组件
│   └── BusinessArchitectureEntry.swift       # L2 公开能力入口
├── Infrastructure/                           # L3：底层技术能力
│   ├── Configuration/                        # 项目配置
│   │   └── AppEnvironment.swift              # 项目环境配置与第三方常量
│   ├── Networking/                           # URLSession、HTTPClient、Endpoint、网络错误
│   ├── Persistence/                          # SwiftData、UserDefaults 底层适配
│   ├── SecureStorage/                        # Keychain 封装
│   ├── FileSystem/                           # 文件系统和临时目录适配
│   ├── Observability/                        # 日志、监控、崩溃和埋点
│   ├── CocoaPods/                            # CocoaPods SDK 适配边界
│   │   ├── FirebaseAdapter.swift             # Firebase SDK 适配
│   │   ├── KingfisherAdapter.swift           # Kingfisher SDK 适配
│   │   └── LottieAdapter.swift               # Lottie SDK 适配
│   └── InfrastructureEntry.swift             # L3 协议和工厂入口
├── Tests/                                    # 单元测试和集成测试
│   ├── BusinessTests/                        # L1 ViewModel、Service、Validator 测试
│   ├── BusinessArchitectureTests/           # L2 能力测试
│   └── InfrastructureTests/                 # L3 网络、存储和 Pod Adapter 测试
├── ProductUITests/                           # XCUITest 完整用户流程测试
├── Configurations/                           # Debug 和 Release 配置
│   ├── Debug.xcconfig                       # 开发环境配置
│   └── Release.xcconfig                     # 发布环境配置
├── Scripts/                                  # 构建、代码生成和 CI 脚本
└── Docs/                                     # 架构决策和业务文档
```

## 4. 导入和依赖规则

| 规则 | 固定要求 |
| --- | --- |
| L0 | 只能组装 L1、L2、L3，不实现业务规则 |
| L1 | 只能依赖 L2、L3；禁止 import App 和其他业务内部文件 |
| L2 | 只能依赖 L3；禁止依赖具体业务模块 |
| L3 | 只能依赖系统 SDK、CocoaPods 和其他 L3 模块 |
| CocoaPods | Pod 只能由 `Infrastructure/CocoaPods/` 的 Adapter 导入 |
| View | 只能调用本业务 ViewModel 和 L2 公开能力 |
| ViewModel | 只负责状态、事件和业务 Service 调用，不直接访问 Pod、网络和数据库 |
| 横向协作 | L1 业务由 L0 AppCoordinator 编排；L2 能力通过协议和参数协作 |
| 第三方类型 | Pod 的类型、错误和回调不得暴露到 L1、L2 |

## 5. 典型调用路径

```text
读取：L0 AppRootView → L1 Business ViewModel → L1 Service → L2 Capability → L3 HTTPClient/Store
写入：L1 View → L1 ViewModel → L1 Service → L2 Service 或 L3 Repository → L3 Persistence/Keychain
启动：ProductApp → AppContainer → L3 Infrastructure → L2 Business Architecture → L1 Business Entry
```

## 6. 代码放置决策表

| 判断问题 | 唯一放置位置 |
| --- | --- |
| 是否为 App 入口、根导航、Deep Link、生命周期或全局组装？ | `App/`，L0 Application |
| 是否实现登录、项目、用户、设置等具体业务场景？ | `Business/<业务>/`，L1 Business |
| 是否被两个及以上业务稳定复用且包含账户、组织、权限等业务语义？ | `BusinessArchitecture/`，L2 Business Architecture |
| 是否为 URLSession、SwiftData、UserDefaults、Keychain、日志或 CocoaPods SDK 适配？ | `Infrastructure/`，L3 Infrastructure |
| 是否为某业务专属 View、ViewModel、Model、Service 或 Validator？ | 对应 `Business/<业务>/` 子目录 |
| 是否为 CocoaPods 第三方库封装？ | `Infrastructure/CocoaPods/` |
| 仍无法归属？ | 拆分混合职责后放入最接近的层，禁止创建顶层杂物目录 |

## 7. CocoaPods 使用规则

`Podfile` 是唯一的 Pod 依赖声明文件，固定放在项目根目录。

```ruby
platform :ios, '17.0'
use_frameworks!

target 'Product' do
  pod 'Alamofire'
  pod 'Kingfisher'
  pod 'Lottie'
  pod 'Firebase/Crashlytics'
end
```

| 规则 | 要求 |
| --- | --- |
| 添加依赖 | 先确定所属 L3 能力，再修改 Podfile，并创建对应 Adapter |
| 版本管理 | Podfile.lock 必须提交 Git |
| 安装命令 | 使用 `pod install` 安装；升级依赖使用 `pod update` |
| 工程打开 | 使用 Product.xcworkspace，禁止直接打开 Product.xcodeproj 编译 Pods 项目 |
| Pods 目录 | 禁止直接修改，所有扩展通过 Infrastructure Adapter 完成 |
| 业务封装 | 不允许把 Pod 类型和错误直接暴露给 L1 或 L2 |

## 8. 测试和质量检查

| 测试类型 | 固定位置 | 验证范围 |
| --- | --- | --- |
| L1 单元测试 | `Tests/BusinessTests/` | ViewModel、Service、Validator 和业务状态 |
| L2 单元测试 | `Tests/BusinessArchitectureTests/` | 账户、会话、权限和组织能力 |
| L3 单元测试 | `Tests/InfrastructureTests/` | HTTPClient、存储、Keychain 和 Pod Adapter |
| UI 自动化测试 | `ProductUITests/` | 登录、核心业务流程、错误重试和 Deep Link |
| 依赖检查 | CI 脚本 | 四层反向依赖、Pod 直接导入和未使用依赖 |

## 9. 禁止的目录设计

| 禁止项 | 固定替代方式 |
| --- | --- |
| 在 App 中实现业务规则 | 放入 L1 Business |
| 在 View 中直接调用 Pod SDK | 通过 L3 Adapter 和上层协议调用 |
| 创建顶层 Services、Managers、Helpers 大集合 | 按业务放 L1，按能力放 L2，按技术放 L3 |
| L1 直接依赖另一个 L1 业务 | 由 L0 AppCoordinator 编排 |
| L2 直接依赖 L1 | 通过参数和协议协作 |
| L3 引入业务模型或业务判断 | 业务模型和规则放 L1/L2 |
| 直接修改 Pods 目录 | 创建 Infrastructure Adapter |

生成新代码时，必须先使用代码放置决策表确定层级，再确定具体业务目录或基础设施目录。
