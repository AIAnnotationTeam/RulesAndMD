# Flutter 项目的常用第三方库

本清单面向中大型 Flutter 项目，采用与 [Flutter 架构模板](flutter-架构.md) 一致的 Feature-first、MVVM、Domain/Data 分层。每类能力固定一个主要实现，不并列引入承担相同职责的状态管理、路由和数据库框架。基础依赖进入工程基线；业务能力库仅在对应功能存在时安装。

## 选型与核查口径

| 项目 | 口径 |
| --- | --- |
| 核查日期 | 2026-10-21 |
| 使用情况 | 结合 pub.dev 下载指标、GitHub 关注度和 Flutter 官方维护背景判断；下载次数、Star 均不等于独立用户数 |
| 社区维护 | 核查仓库是否归档及最近推送记录；推送可能来自机器人，不能单独证明维护质量 |
| 数据限制 | 本次 pub.dev 多数请求超时，仅对成功取得的指标给出数值；其余不填充下载量、排名或未经核实的最新版本 |
| 平台范围 | Flutter 包不等于全平台能力一致；插件必须在目标平台核对官方平台表、最低系统版本和原生配置 |
| 安装原则 | 基础能力统一选型；业务能力按实际功能安装；代码生成器和测试工具放入 `dev_dependencies` |
| 版本管理 | 本文不写浮动版本号；项目安装时解析兼容稳定版本，应用提交 `pubspec.lock`，升级执行回归测试 |

## 状态管理与依赖注入工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [flutter_riverpod](https://pub.dev/packages/flutter_riverpod) | 提供 Provider、Notifier、AsyncNotifier、异步状态和依赖覆盖，统一页面状态及依赖注入。 | 基础依赖；页面状态归属对应 Feature，禁止同时引入 Bloc、GetX 或另一套 DI 容器。 |
| [riverpod_annotation](https://pub.dev/packages/riverpod_annotation) | Riverpod 代码生成注解，声明 Provider 与 Notifier。 | 运行依赖；与生成器使用相互兼容的版本。 |
| [riverpod_generator](https://pub.dev/packages/riverpod_generator) | 生成类型安全的 Provider 定义，减少样板代码。 | 开发依赖；由 `build_runner` 执行。 |

## 路由与导航工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [go_router](https://pub.dev/packages/go_router) | Flutter 团队维护的声明式路由，支持深链、重定向、嵌套路由和多分支导航。 | 基础依赖；根路由位于 `app/navigation`，Feature 暴露自己的路由入口；不混用另一套路由框架。 |

## 网络通信工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [dio](https://pub.dev/packages/dio) | HTTP 客户端，支持拦截器、取消、上传下载、超时和进度回调。 | 基础依赖；公共客户端在 `core/network`，接口访问在 Feature Data 层。 |
| [connectivity_plus](https://pub.dev/packages/connectivity_plus) | Flutter Community 维护的网络连接类型监听插件。 | 网络感知功能；连接到 Wi-Fi 不代表互联网可达，请求成功与否仍由网络层判断。 |

## 数据建模与序列化工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [json_annotation](https://pub.dev/packages/json_annotation) | JSON 序列化注解，配置字段名称、默认值和转换规则。 | 基础运行依赖；用于 Data 层 DTO。 |
| [json_serializable](https://pub.dev/packages/json_serializable) | 生成 JSON 编解码代码，避免手工维护映射。 | 基础开发依赖；不让 API DTO 穿透至 Domain 或页面。 |
| [freezed_annotation](https://pub.dev/packages/freezed_annotation) | Freezed 注解，用于不可变模型和联合类型声明。 | 模型生成依赖；适用于 UiState、DTO 和值对象，不承担状态管理。 |
| [freezed](https://pub.dev/packages/freezed) | 生成不可变对象、`copyWith`、值相等和联合类型，配合 JSON 工具处理模型。 | 开发依赖；使用 Freezed 的纯 Dart 类型可以进入 Domain，禁止夹带 Flutter、HTTP 或数据库类型。 |

## 本地存储工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [drift](https://pub.dev/packages/drift) | 基于 SQLite 的响应式持久化框架，提供类型安全查询、事务、迁移和数据流。 | 基础数据库；底层连接在 `core/database`，业务访问经 Repository；Web 需要单独配置 WASM/Worker。 |
| [drift_dev](https://pub.dev/packages/drift_dev) | Drift 数据库和查询代码生成工具。 | 开发依赖；随 Drift 使用。 |
| [shared_preferences](https://pub.dev/packages/shared_preferences) | Flutter 官方插件，保存应用设置和少量键值配置。 | 基础依赖；不保存密码、Token、关键交易数据或大体量业务记录。 |
| [flutter_secure_storage](https://pub.dev/packages/flutter_secure_storage) | 通过各平台安全存储机制保存凭证等敏感数据。 | 基础依赖；统一封装在 `core/secure_storage`；Web 实现不具备原生 Keychain 的同等安全边界。 |
| [path_provider](https://pub.dev/packages/path_provider) | Flutter 官方插件，获取应用文档、缓存和临时目录。 | 本地文件功能；没有 Web 文件目录实现，Web 通过独立适配访问浏览器存储。 |

## 界面与图片工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [cached_network_image](https://pub.dev/packages/cached_network_image) | 网络图片加载工具，提供占位、错误展示和缓存集成。 | 图片能力基线；Web 的缓存行为与原生磁盘缓存不同，不能假设平台一致。 |
| [flutter_svg](https://pub.dev/packages/flutter_svg) | Flutter 团队维护的 SVG 渲染工具，用于矢量图标与插画。 | SVG 资源功能；遵循库支持的 SVG 子集，复杂滤镜等效果需验证。 |
| [lottie](https://pub.dev/packages/lottie) | Lottie 动画播放器，用于引导、加载、空状态和业务反馈动画。 | 动画功能；控制素材体积、帧率与同时播放数量。 |

Material 3 组件、响应式布局、下拉刷新和普通隐式/显式动画直接使用 Flutter SDK，不为 SDK 已覆盖的基础能力重复引入组件库。

## 表单与国际化工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [flutter_form_builder](https://pub.dev/packages/flutter_form_builder) | 统一管理复杂表单字段、初始值、校验、保存和重置。 | 多字段业务表单；提交与业务规则仍由 ViewModel 和 UseCase 执行。 |
| [form_builder_validators](https://pub.dev/packages/form_builder_validators) | 提供可组合的常用字段校验器和校验文案本地化。 | 与表单库配套；客户端校验不替代服务端规则。 |
| [intl](https://pub.dev/packages/intl) | Dart 国际化基础库，用于日期、数字、货币格式化及消息处理。 | 国际化功能；文案仍使用 Flutter `gen_l10n` 与 ARB；版本约束与 `flutter_localizations` 对齐。 |

## 文件、权限与系统交互工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [image_picker](https://pub.dev/packages/image_picker) | Flutter 官方图片和视频选择插件，提供相册选择和受支持平台的拍摄入口。 | 媒体选择功能；桌面拍摄能力需要额外平台适配。 |
| [file_picker](https://pub.dev/packages/file_picker) | 通过系统文件选择器选择文件和目录，支持扩展名过滤与多选。 | 附件上传和文件导入；浏览器返回内容不等于本地可直接访问的文件路径。 |
| [permission_handler](https://pub.dev/packages/permission_handler) | 统一查询和请求受支持平台上的权限。 | 权限功能；声明原生权限及用途说明，按功能触发授权。 |
| [url_launcher](https://pub.dev/packages/url_launcher) | Flutter 官方插件，打开网页、电话、邮件与平台 URL Scheme。 | 外部跳转；处理未安装目标应用和启动失败。 |
| [share_plus](https://pub.dev/packages/share_plus) | Flutter Community 维护的系统分享插件，用于文本、链接和文件分享。 | 分享功能；可用的分享内容和能力随平台变化。 |
| [webview_flutter](https://pub.dev/packages/webview_flutter) | Flutter 官方 WebView 插件，提供网页嵌入、导航控制和 JavaScript 交互。 | Android、iOS、macOS 的网页嵌入；不作为 Web、Windows、Linux 通用实现，限制加载域和 JS 桥能力。 |

## 音视频工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [video_player](https://pub.dev/packages/video_player) | Flutter 官方视频播放插件，提供视频控制、进度与播放状态。 | 视频功能；默认平台实现不能覆盖所有桌面目标，播放格式和自动播放规则按平台验证。 |
| [just_audio](https://pub.dev/packages/just_audio) | 音频播放器，支持播放列表、循环、速度调整和音频流。 | 音频功能；后台播放与音频会话需要独立配置，部分桌面平台需对应实现包。 |

## 通知与推送工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [flutter_local_notifications](https://pub.dev/packages/flutter_local_notifications) | 本地通知插件，支持通知展示、计划任务和点击回调。 | 本地提醒；不是服务端推送通道，平台权限、时区和调度限制分别处理。 |
| [firebase_core](https://pub.dev/packages/firebase_core) | FlutterFire 初始化入口，连接 Firebase 项目与原生配置。 | 集成 Firebase 能力时引入；环境配置在启动阶段装配。 |
| [firebase_messaging](https://pub.dev/packages/firebase_messaging) | FlutterFire 的 FCM 推送客户端，处理设备 Token、前后台消息和通知交互。 | FCM 推送业务；依赖 Firebase Core；中国大陆 Android 设备的服务可达性与厂商通道另行落实，不能视为全渠道覆盖。 |

## 订阅与付费工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [purchases_flutter](https://pub.dev/packages/purchases_flutter) | RevenueCat 官方 SDK，统一商店订阅、购买、恢复购买和用户权益同步。 | 商店内购与订阅业务；这是带服务端服务及计费规则的产品集成，不是通用第三方支付 SDK；正式权限校验在可信服务端完成。 |

## 日志与线上监控工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [talker](https://pub.dev/packages/talker) | 结构化日志、异常记录和日志事件管理。 | 基础日志库，封装在 `core/logging`。 |
| [talker_flutter](https://pub.dev/packages/talker_flutter) | Talker 的 Flutter 日志查看界面与集成组件。 | 内部调试入口；生产环境不公开日志面板。 |
| [talker_dio_logger](https://pub.dev/packages/talker_dio_logger) | 将 Dio 请求、响应、异常和耗时接入 Talker。 | 开发排障；脱敏 Authorization、Cookie、个人信息及请求正文。 |
| [sentry_flutter](https://pub.dev/packages/sentry_flutter) | Sentry 官方 Flutter SDK，采集线上异常、崩溃与性能遥测。 | 线上监控；与 Talker 分工：Talker 服务调试日志，Sentry 服务集中诊断；采样和数据采集按环境配置。 |

## 代码生成与质量工具

| 第三方库 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [build_runner](https://pub.dev/packages/build_runner) | Dart 构建及代码生成基础工具，统一运行 Riverpod、JSON、Freezed、Drift 生成任务。 | 基础开发依赖；生成文件禁止手工修改。 |
| [very_good_analysis](https://pub.dev/packages/very_good_analysis) | Very Good Ventures 维护的 Dart/Flutter 静态分析规则集。 | 基础开发依赖；由 `analysis_options.yaml` 引入，CI 执行 `flutter analyze`。 |

## 自动化测试工具

| 工具 | 简要介绍 | 使用边界 |
| --- | --- | --- |
| [mocktail](https://pub.dev/packages/mocktail) | 无需代码生成的 Mock 工具，替换 Repository、Service 和外部依赖。 | 基础开发依赖；结合 Provider 覆盖测试 ViewModel 和 UseCase。 |
| `flutter_test`（Flutter SDK） | 单元测试、Widget 测试、交互断言及 Golden 图像比较。 | SDK 开发依赖；Golden 测试使用 `matchesGoldenFile`，固定 CI 系统、字体和渲染配置。 |
| `integration_test`（Flutter SDK） | 在目标设备上验证跨页面业务流程。 | SDK 开发依赖；覆盖登录、数据同步和关键交易流程。 |

### 与现有架构模板的测试差异

| 项目 | 本清单处理 |
| --- | --- |
| `golden_toolkit` | 现有架构模板列出该库；其所属 `eBay/flutter_glove_box` 仓库本次查得最近推送为 2024-09-13，未归档，但不足以证明当前活跃维护。本清单不将其列入新项目基线，使用 SDK Golden 测试能力。 |
| SDK 工具归类 | `flutter_test`、`integration_test`、`gen_l10n` 和 Material 3 是官方 SDK 能力，单独标明，不冒充第三方库。 |

## 与架构目录的对应关系

| 目录 | 放入的第三方能力 | 禁止行为 |
| --- | --- | --- |
| `app/bootstrap` | Sentry、日志、数据库初始化与配置装配 | 在 Widget 构建期间重复初始化 SDK。 |
| `app/navigation` | GoRouter 根配置、重定向和导航分支 | 页面到处创建独立 Router。 |
| `features/*/presentation` | Riverpod 页面状态、表单、图片、动画和媒体组件 | 直接读数据库、发 HTTP 请求。 |
| `features/*/domain` | 纯 Dart 业务实体、接口和 UseCase；Freezed 仅生成纯 Dart 类型 | 引入 Dio、Drift、Riverpod 和平台插件。 |
| `features/*/data` | Dio 数据源、JSON DTO、Mapper 和 Repository 实现 | 将 DTO、数据库行或 SDK 对象暴露为领域接口。 |
| `core/network` | Dio 公共客户端、拦截器和连接状态适配 | 根据连接类型直接断言服务可用。 |
| `core/database`、`core/storage`、`core/secure_storage` | Drift、SharedPreferences、安全存储的基础封装 | 在通用基础设施中导入 Feature。 |
| `core/platform` | 文件、权限、分享、通知和商店 SDK 的平台适配 | 在业务代码中分散平台判断和原生异常处理。 |
| `core/logging` | Talker、Dio 日志和 Sentry 适配 | 记录密钥、完整凭证和用户敏感内容。 |

## 联网核查记录

下列记录来自本次实际响应。GitHub 数值为仓库级 Star，多包仓库不能拆分为单包使用人数；最近推送统一按 API 返回值的 UTC 日期记录，不能解读为最新发布日。

| 项目 | 指标 | 查询结果 | 来源 |
| --- | --- | --- | --- |
| `flutter_riverpod` | pub.dev 最近 30 天下载次数 / Likes | 3,775,804 / 2,915 | [pub.dev Score API](https://pub.dev/api/packages/flutter_riverpod/score) |
| `riverpod_annotation` | pub.dev 最近 30 天下载次数 / Likes | 1,288,644 / 181 | [pub.dev Score API](https://pub.dev/api/packages/riverpod_annotation/score) |
| Riverpod | GitHub Star / 最近推送 / 归档 | 7,409 / 2026-10-20 / 否 | [仓库](https://github.com/rrousselGit/riverpod) |
| Dio | GitHub Star / 最近推送 / 归档 | 12,842 / 2026-10-19 / 否 | [仓库](https://github.com/cfug/dio) |
| Drift | GitHub Star / 最近推送 / 归档 | 3,294 / 2026-10-20 / 否 | [仓库](https://github.com/simolus3/drift) |
| Freezed | GitHub Star / 最近推送 / 归档 | 2,198 / 2026-10-17 / 否 | [仓库](https://github.com/rrousselGit/freezed) |
| Flutter 官方 Packages | GitHub Star / 最近推送 / 归档 | 5,324 / 2026-10-21 / 否 | [仓库](https://github.com/flutter/packages) |
| FlutterFire | GitHub Star / 最近推送 / 归档 | 9,261 / 2026-10-21 / 否 | [仓库](https://github.com/firebase/flutterfire) |
| Cached Network Image | GitHub Star / 最近推送 / 归档 | 2,590 / 2026-09-30 / 否 | [仓库](https://github.com/Baseflow/flutter_cached_network_image) |
| Flutter Secure Storage | GitHub Star / 最近推送 / 归档 | 1,286 / 2026-10-21 / 否 | [仓库](https://github.com/juliansteenbakker/flutter_secure_storage) |
| Flutter Community Plus Plugins | GitHub Star / 最近推送 / 归档 | 1,876 / 2026-10-14 / 否 | [仓库](https://github.com/fluttercommunity/plus_plugins) |
| Flutter Form Builder | GitHub Star / 最近推送 / 归档 | 1,603 / 2026-08-15 / 否 | [仓库](https://github.com/flutter-form-builder-ecosystem/flutter_form_builder) |
| Flutter Local Notifications | GitHub Star / 最近推送 / 归档 | 2,671 / 2026-10-13 / 否 | [仓库](https://github.com/MaikuB/flutter_local_notifications) |
| Just Audio | GitHub Star / 最近推送 / 归档 | 1,220 / 2026-06-29 / 否 | [仓库](https://github.com/ryanheise/just_audio) |
| Talker | GitHub Star / 最近推送 / 归档 | 844 / 2026-10-20 / 否 | [仓库](https://github.com/Frezyx/talker) |
| Sentry Dart/Flutter | GitHub Star / 最近推送 / 归档 | 874 / 2026-10-20 / 否 | [仓库](https://github.com/getsentry/sentry-dart) |
| RevenueCat Flutter | GitHub Star / 最近推送 / 归档 | 720 / 2026-10-21 / 否 | [仓库](https://github.com/RevenueCat/purchases-flutter) |
| Mocktail | GitHub Star / 最近推送 / 归档 | 702 / 2026-04-12 / 否 | [仓库](https://github.com/felangel/mocktail) |
| Very Good Analysis | GitHub Star / 最近推送 / 归档 | 477 / 2026-10-13 / 否 | [仓库](https://github.com/VeryGoodOpenSource/very_good_analysis) |

## 选型结论

中大型项目统一使用 Riverpod、go_router、Dio、Drift、JSON 代码生成和 Talker 作为基础能力，线上诊断接入 Sentry。系统能力采用 Flutter 官方插件或持续维护的成熟社区插件，业务 SDK 经平台适配和 Repository 接入。项目需要的能力才进入依赖清单，避免把本目录当成需要全部安装的套餐。
