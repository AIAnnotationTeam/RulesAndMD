# Flutter 中大型项目架构模板

本文是 Flutter 中大型项目的固定工程模板。所有业务代码按 Feature-first 组织，Feature 内部采用 Flutter 官方架构中的 UI 层、Domain 层和 Data 层；UI 层使用 MVVM，数据访问使用 Repository + Service。目录职责和依赖方向均已固定，新增代码必须按本文归位。

## 1. 固定技术栈

| 能力 | 固定技术 | 使用边界 |
| --- | --- | --- |
| UI | Flutter + Material 3 | 页面、组件和主题统一使用 Flutter Widget 与 Material 3 |
| 状态管理 | Riverpod + `riverpod_annotation` + `riverpod_generator` | Provider 按 Feature 归属，负责状态和依赖注入 |
| 页面路由 | `go_router` | 根路由集中注册，Feature 提供路由入口 |
| 网络请求 | Dio | 统一由 `core/network` 封装拦截器、错误和超时 |
| JSON 序列化 | `json_annotation` + `json_serializable` | Data 层 DTO 使用代码生成 |
| 本地数据库 | Drift | 结构化离线数据和缓存使用 Drift |
| 轻量存储 | `shared_preferences` | 非敏感配置和简单键值使用 |
| 敏感存储 | `flutter_secure_storage` | Token、密钥和凭证使用 |
| 图片缓存 | `cached_network_image` | 网络图片统一使用缓存组件 |
| 国际化 | Flutter `gen_l10n` | 文案放在 `lib/l10n`，禁止在 Widget 中硬编码多语言文案 |
| 日志与错误 | Talker | 统一记录调试日志、网络日志和异常信息 |
| 代码生成 | `build_runner` | 生成 Riverpod、JSON、Drift 等文件 |
| 测试 | `flutter_test` + `mocktail` + `integration_test` + `golden_toolkit` | 单元、Widget、集成和视觉回归测试 |
| 静态检查 | `very_good_analysis` | 统一 Dart lint、格式和复杂度约束 |

## 2. 总体架构层级

| 层级 | 固定目录 | 职责 | 允许依赖 |
| --- | --- | --- | --- |
| Application | `lib/app` | 启动、环境、根路由、全局 Provider、主题和跨 Feature 流程编排 | `core`、`shared`、各 Feature 的公开入口 |
| Presentation | `lib/features/*/presentation` | 页面、Widget、ViewModel/Notifier、UI 状态和 Feature 路由 | 本 Feature 的 `domain`、`core`、`shared` |
| Domain | `lib/features/*/domain` | Entity、Repository 接口、UseCase 和业务规则 | `shared` 中的稳定基础类型 |
| Data | `lib/features/*/data` | DTO、Service、DataSource、Repository 实现、Mapper 和持久化 | 本 Feature 的 `domain`、`core`、第三方数据 SDK |
| Core | `lib/core` | 网络、数据库、存储、平台适配、设计系统、错误和日志 | Flutter SDK 与第三方基础库，不依赖 Feature |
| Shared | `lib/shared` | 真正跨业务复用的无业务模型、枚举和工具 | Dart/Flutter 基础库 |

依赖方向固定为：

```text
Application ──> Feature Presentation ──> Feature Domain
     │                    │                    ▲
     ├──────────────> Core ───────────────> Feature Data
     └──────────────> Shared <───────────────┘
Feature Data ──实现──> Feature Domain 的 Repository 接口
```

Domain 不依赖 Data、Flutter 页面、Dio 或 Drift。Data 通过依赖倒置实现 Domain 的 Repository 接口。View 只消费 UI 状态并发送事件，ViewModel/Notifier 调用 UseCase。

## 3. 标准项目目录

```text
project_name/
├── android/                         # Android 宿主工程、Gradle 配置和原生插件
├── ios/                             # iOS 宿主工程、Xcode 配置和原生插件
├── web/                             # Web 宿主入口、构建配置和 Web 资源
├── macos/                           # macOS 宿主工程
├── windows/                         # Windows 宿主工程
├── linux/                           # Linux 宿主工程
├── assets/                          # 由 pubspec.yaml 声明的静态资源
│   ├── images/                      # 图片和插画
│   ├── icons/                       # SVG、图标和状态图
│   ├── fonts/                       # 项目字体文件
│   └── animations/                  # Lottie 等动画资源
├── lib/
│   ├── main.dart                    # 生产入口：调用 bootstrap 后运行 App
│   ├── main_development.dart        # 开发环境入口
│   ├── main_staging.dart            # 测试环境入口
│   ├── app/                         # 应用级编排，不承载具体业务规则
│   │   ├── app.dart                  # MaterialApp.router 根组件
│   │   ├── bootstrap/                # 启动初始化、异常捕获和依赖装配
│   │   ├── config/                   # 环境变量、构建 flavor 和应用常量
│   │   ├── navigation/               # GoRouter 实例、认证重定向和根路由
│   │   ├── providers/                # 仅应用级 Provider，如环境和会话容器
│   │   └── theme/                    # Material 3 ThemeData、颜色、字体和尺寸
│   ├── features/                     # 按业务能力拆分的功能模块
│   │   ├── authentication/           # 登录、注册、Token 和会话
│   │   ├── user/                     # 用户资料和账户设置
│   │   ├── project/                  # 项目领域业务
│   │   ├── notification/             # 站内通知和推送状态
│   │   └── settings/                 # 应用设置
│   ├── core/                         # 跨 Feature 的技术基础设施
│   │   ├── design_system/            # 设计 Token、主题扩展和基础视觉组件
│   │   ├── widgets/                  # 无业务语义的通用 Widget
│   │   ├── network/                  # Dio 客户端、拦截器、请求结果和网络异常
│   │   ├── database/                 # Drift 数据库、表、迁移和 DAO 基础设施
│   │   ├── storage/                  # SharedPreferences 封装
│   │   ├── secure_storage/           # Secure Storage 封装
│   │   ├── logging/                  # Talker 初始化、日志格式和上报适配
│   │   ├── config/                   # 基础配置解析和运行时开关
│   │   ├── errors/                   # AppException、Failure 和错误映射
│   │   ├── extensions/               # 无业务含义的 Dart/Flutter 扩展
│   │   ├── platform/                 # MethodChannel、文件、权限等平台适配
│   │   └── testing/                  # 测试工厂、Fake 和公共测试辅助类
│   ├── shared/                       # 跨业务且稳定的无业务代码
│   │   ├── models/                   # 通用分页、结果包装等基础模型
│   │   ├── enums/                    # 跨 Feature 的基础枚举
│   │   └── utils/                    # 纯函数和通用工具
│   └── l10n/                         # Flutter gen-l10n ARB 文件和本地化生成入口
├── test/                             # 单元测试、Widget 测试和 Golden 测试
│   ├── features/                     # 按 Feature 镜像 lib/features
│   ├── core/                         # Core 测试
│   └── fixtures/                     # 测试固定数据
├── integration_test/                 # 真机或模拟器端到端测试
├── tool/                             # 代码生成、检查和 CI 辅助脚本
├── analysis_options.yaml              # very_good_analysis 规则入口
├── build.yaml                        # build_runner 生成配置
├── pubspec.yaml                      # 依赖、资源、字体和生成配置
├── pubspec.lock                      # 应用锁定依赖版本
└── README.md                         # 项目运行、构建和目录说明
```

宿主目录由 Flutter 工具生成并分别维护平台工程；业务 Dart 代码只放在 `lib`、`test` 和 `integration_test`。不得把业务逻辑写入宿主目录或 `main.dart`。

## 4. Feature 内部模板

每个业务 Feature 必须使用下列结构：

```text
lib/features/<feature>/
├── <feature>.dart                    # Feature 对外唯一入口，导出路由或装配函数
├── presentation/                     # UI 和页面状态
│   ├── pages/                         # 路由页面，一个页面一个文件
│   ├── widgets/                       # 仅服务本 Feature 的业务组件
│   ├── providers/                     # Riverpod Provider、Notifier 和依赖装配
│   ├── states/                        # 不可变 UI State、枚举和状态派生
│   └── routes/                        # Feature 路由声明和路由参数解析
├── domain/                            # 与 UI、网络和存储无关的业务核心
│   ├── entities/                      # 业务实体和值对象
│   ├── repositories/                  # Repository 抽象接口
│   └── use_cases/                     # 一个文件表达一个业务意图
└── data/                              # 外部数据源和转换
    ├── datasources/                   # Remote Service、Local DataSource
    ├── models/                        # API DTO、Drift 表映射模型
    ├── mappers/                       # DTO/数据库模型与 Domain Entity 转换
    └── repositories/                  # Repository 接口的具体实现
```

### 4.1 各文件的固定职责

| 文件类型 | 必须承担的职责 | 禁止承担的职责 |
| --- | --- | --- |
| Page/View | 订阅 ViewModel 状态、渲染界面、发送用户事件 | 直接调用 Dio、Drift、SharedPreferences 或拼接业务规则 |
| ViewModel/Notifier | 维护加载/成功/失败状态，调用 UseCase，处理用户事件 | 直接解析 JSON、编写 SQL 或访问平台 API |
| UiState | 表达页面渲染所需的不可变状态 | 持有 Dio、数据库连接或 Widget 引用 |
| UseCase | 表达单一业务意图，组合领域规则 | 依赖具体 Repository 实现或 Flutter BuildContext |
| Repository 接口 | 定义领域需要的数据能力 | 暴露 DTO、HTTP 响应或数据库表 |
| Repository 实现 | 协调远程和本地数据源，处理缓存策略 | 被 Page 直接依赖 |
| Service/DataSource | 封装一个外部数据源 | 编排跨数据源业务流程 |
| DTO/数据库模型 | 适配 API 或数据库结构 | 泄漏到 Presentation 或 Domain |
| Mapper | 完成 DTO/数据库模型与 Entity 的转换 | 发起网络请求或修改状态 |

## 5. 依赖与命名规则

| 规则 | 固定要求 |
| --- | --- |
| Feature 隔离 | Feature 之间禁止直接导入内部文件；跨 Feature 流程由 `app` 编排，公共类型移入 `shared` |
| Core 隔离 | `core` 禁止导入任何 `features` 文件 |
| Domain 纯净 | Domain 禁止依赖 Dio、Drift、Riverpod、Widget 和平台 API |
| Provider 归属 | Provider 放入所属 Feature 的 `presentation/providers`；应用级 Provider 才能放入 `app/providers` |
| 路由归属 | 根 GoRouter 放在 `app/navigation`；Feature 在 `presentation/routes` 暴露路由注册函数 |
| 平台调用 | 所有 MethodChannel、权限和文件系统调用经 `core/platform` 适配 |
| 数据转换 | Data 层返回 Domain Entity；DTO 和数据库模型不跨越 Data 层 |
| 生成文件 | `.g.dart`、`.freezed.dart`、Drift 生成文件由 `build_runner` 产生，禁止手工编辑 |
| 命名 | 文件使用 `snake_case.dart`；类使用 `UpperCamelCase`；Provider 使用 `...Provider`；UseCase 使用动词短语 |
| 导出 | 每个 Feature 只通过 `<feature>.dart` 暴露公共入口，禁止导出 Data 内部实现 |

## 6. 代码放置决策表

| 需求 | 放置位置 |
| --- | --- |
| 某个页面的 UI | `features/<feature>/presentation/pages` |
| 只被一个 Feature 使用的业务组件 | `features/<feature>/presentation/widgets` |
| 多个 Feature 使用且没有业务语义的组件 | `core/widgets` 或 `core/design_system` |
| 页面加载、分页、提交状态 | `features/<feature>/presentation/states` 与 ViewModel |
| 登录业务规则 | `features/authentication/domain/use_cases` |
| REST/GraphQL 请求 | 对应 Feature 的 `data/datasources`，底层客户端在 `core/network` |
| API 返回结构 | 对应 Feature 的 `data/models` |
| 本地缓存策略 | 对应 Feature 的 Repository 实现；数据库能力在 `core/database` |
| Token 读写 | `core/secure_storage`，由 authentication Data 层调用 |
| 全局主题和颜色 | `app/theme` 与 `core/design_system` |
| 环境地址和 flavor | `app/config` |
| 通用分页模型 | `shared/models` |
| 纯 Dart 日期、格式化工具 | `shared/utils` 或 `core/extensions` |

## 7. 数据流与错误处理

```text
用户操作
  ↓
Page/View ──事件──> Riverpod ViewModel/Notifier
  ↓                         ↓
渲染 UiState <──结果── UseCase
                              ↓
                    Domain Repository 接口
                              ↓
                 Data Repository 实现
                    ↙                  ↘
             Remote Service       Local DataSource
                    ↓                  ↓
                 DTO/模型 ──Mapper──> Domain Entity
```

Data 层把 Dio、Drift 和平台异常转换为 `core/errors` 中的 Failure；UseCase 只返回 Entity 或 Failure；ViewModel 将 Failure 映射为 UiState 中的用户可见错误。日志写入 Talker，用户界面不展示原始堆栈和敏感信息。

## 8. 测试边界

| 测试类型 | 测试对象 | 目录 |
| --- | --- | --- |
| Domain 单元测试 | Entity、UseCase、业务规则 | `test/features/<feature>/domain` |
| Data 单元测试 | Mapper、Repository、DataSource | `test/features/<feature>/data` |
| ViewModel 测试 | 状态转换、事件处理、错误状态 | `test/features/<feature>/presentation` |
| Widget 测试 | 页面渲染、交互和可访问性 | `test/features/<feature>/presentation` |
| Golden 测试 | 设计系统和关键页面视觉快照 | `test/core`、`test/features` |
| 集成测试 | 登录、核心业务流程和真实平台能力 | `integration_test` |

测试通过 Provider 覆盖注入 Fake 或 Mock 数据源；测试不得依赖真实生产接口和真实用户数据。

## 9. 禁止的目录设计

| 禁止设计 | 原因 |
| --- | --- |
| `lib/screens`、`lib/controllers`、`lib/services` 作为全局平铺目录 | 业务边界消失，代码归属不明确 |
| `lib/models` 存放所有模型 | DTO、Entity 和数据库模型混杂，层间耦合 |
| 页面直接调用 Dio 或数据库 | 绕过 UseCase 和 Repository，无法替换数据源 |
| 创建无业务边界的全局 `providers` | 状态生命周期和所有权不可追踪 |
| Feature 互相导入内部目录 | 形成循环依赖和隐式耦合 |
| `utils` 收纳任意代码 | 工具职责失控，无法判断复用边界 |
| 在 `main.dart` 编写业务初始化和业务流程 | 入口无法测试，环境配置与业务耦合 |

## 10. 初始化与质量门禁

项目启动顺序固定为：读取环境配置 → 初始化日志 → 初始化存储和数据库 → 创建 Provider 容器 → 创建 GoRouter → 运行 `App`。CI 必须依次执行 `dart format --output=none --set-exit-if-changed .`、`flutter analyze`、`flutter test` 和集成测试。

代码生成统一执行：

```bash
dart run build_runner build --delete-conflicting-outputs
```

## 11. 联网调研依据

| 来源 | 关键结论 | 对本模板的落地 |
| --- | --- | --- |
| [Flutter App architecture guide](https://docs.flutter.dev/app-architecture/guide) | Flutter 官方将应用划分为 UI、Data；复杂应用加入 Domain；UI 由 Views 和 ViewModels 构成，Data 由 Repositories 和 Services 构成 | 固定为 Presentation（MVVM）+ Domain + Data，并以 Repository 作为数据唯一真实来源 |
| [Flutter state management options](https://docs.flutter.dev/data-and-backend/state-mgmt/options) | Flutter 不强制唯一状态管理库，状态方案应满足可测试、可维护和依赖注入 | 中大型项目固定 Riverpod，统一 Provider 生命周期和测试替换 |
| [Flutter navigation and routing](https://docs.flutter.dev/ui/navigation) | 复杂应用需要声明式路由和深链支持 | 使用 `go_router` 集中管理根路由和重定向 |
| [Flutter internationalization](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization) | 官方 `gen_l10n` 通过 ARB 和代码生成提供本地化 | 文案统一放入 `lib/l10n`，禁止页面硬编码多语言文本 |

上述官方资料确认了分层职责；具体第三方库版本由项目初始化时锁定在 `pubspec.lock`，升级必须经过测试和兼容性验证。
