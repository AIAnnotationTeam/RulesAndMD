# React Native 中大型项目架构规范

本规范适用于 TypeScript 裸 React Native 中大型项目，覆盖 iOS、Android。导航固定使用 React Navigation，服务端状态固定使用 TanStack Query，客户端共享状态固定使用 Zustand，表单固定使用 React Hook Form 与 Zod，持久化固定使用 MMKV，敏感信息固定使用 Keychain/Keystore。目标是让业务代码内聚、跨端平台能力受控，并让每个新增文件都有明确归属。

采用**按业务域模块化 + 模块内轻量 Clean Architecture**：全局不把所有页面、组件、服务按技术类型平铺；每个业务模块独立拥有展示、应用、领域和数据代码。架构服务于演进，不为文件夹数量 KPI 加班。

## 1. 全局架构层级

| 层级 | 目录 | 职责 | 允许依赖 | 禁止依赖 |
| --- | --- | --- | --- | --- |
| Application | `src/app` | 应用启动、Provider 组装、导航、全局错误处理 | `features`、`shared`、`core` | 具体业务模块的内部实现被反向依赖 |
| Feature | `src/features/<feature>` | 完整业务域：页面、交互、用例、领域规则、数据访问 | 自身、`shared`、`core` | `app`；其他 Feature 的内部文件 |
| Shared | `src/shared` | 跨业务复用的 UI、主题、Hook、工具、通用类型 | `core` | `app`、`features` |
| Core | `src/core` | 无业务含义的平台与技术能力：网络、存储、认证 SDK、原生桥接、配置 | 自身 | `app`、`features`、`shared` |

| Feature 内层 | 目录 | 职责 |
| --- | --- | --- |
| Presentation | `presentation` | Screen、业务组件、视图 Hook、表单与交互状态 |
| Application | `application` | 用例编排、客户端状态、命令；协调领域与数据层 |
| Domain | `domain` | 实体、值对象、业务规则、Repository 抽象；不依赖 React Native 或 HTTP |
| Data | `data` | API、DTO、Repository 实现、Mapper、本地缓存实现 |

## 2. 标准目录模板

业务文件不放入 `app/navigation`，导航层仅声明页面入口和导航结构。

```text
project/
├── android/                                  # Android 原生工程
├── ios/                                      # iOS 原生工程
├── assets/                                   # 图片、字体等打包资源
│   ├── images/
│   ├── icons/
│   └── fonts/
├── src/
│   ├── app/                                  # 应用编排层
│   │   ├── App.tsx                           # 根组件
│   │   ├── providers/
│   │   │   ├── AppProviders.tsx              # Query、状态、主题等 Provider 组装
│   │   │   └── ErrorBoundary.tsx
│   │   ├── navigation/
│   │   │   ├── RootNavigator.tsx             # 根导航器
│   │   │   ├── AuthNavigator.tsx             # 认证区域导航
│   │   │   ├── MainNavigator.tsx             # 登录后主导航
│   │   │   ├── types.ts                      # 路由参数类型
│   │   │   └── linking.ts                    # Deep Link 配置
│   │   ├── bootstrap/
│   │   │   ├── initialize-app.ts             # 启动初始化编排
│   │   │   └── splash-screen.ts
│   │   └── config/
│   │       └── routes.ts                     # 路由名称常量
│   ├── features/                             # 业务域：主要业务代码归属地
│   │   ├── authentication/
│   │   │   ├── presentation/
│   │   │   │   ├── screens/
│   │   │   │   │   ├── LoginScreen.tsx
│   │   │   │   │   └── RegisterScreen.tsx
│   │   │   │   ├── components/
│   │   │   │   └── hooks/
│   │   │   ├── application/
│   │   │   │   ├── use-cases/
│   │   │   │   │   └── sign-in.ts
│   │   │   │   └── state/
│   │   │   │       └── auth.store.ts
│   │   │   ├── domain/
│   │   │   │   ├── entities/
│   │   │   │   └── repositories/
│   │   │   │       └── AuthRepository.ts
│   │   │   ├── data/
│   │   │   │   ├── api/
│   │   │   │   │   └── auth.api.ts
│   │   │   │   ├── repositories/
│   │   │   │   │   └── AuthRepositoryImpl.ts
│   │   │   │   ├── mappers/
│   │   │   │   └── dto/
│   │   │   ├── index.ts                    # Feature 对外公开入口
│   │   │   └── __tests__/
│   │   ├── user/
│   │   ├── project/
│   │   ├── notification/
│   │   └── settings/
│   ├── shared/                               # 无业务含义的跨 Feature 复用代码
│   │   ├── ui/                               # Button、Input、Modal、List 等基础 UI
│   │   ├── components/                       # AppHeader、EmptyState 等组合通用组件
│   │   ├── hooks/                            # useDebounce、useAppState 等通用 Hook
│   │   ├── theme/                            # Token、ThemeProvider、颜色、间距、字体
│   │   ├── types/                            # 通用类型
│   │   └── utils/                            # 日期、金额、字符串等纯函数
│   ├── core/                                 # 技术与平台基础设施
│   │   ├── api/                              # Axios Client、请求拦截器、错误标准化
│   │   ├── auth/                             # 认证 SDK 初始化、Token 刷新与安全存储协调
│   │   ├── config/                           # 环境变量校验、构建配置读取
│   │   ├── native/                           # 原生能力的 TypeScript 包装
│   │   ├── storage/                          # MMKV 与 Keychain/Keystore 适配器
│   │   ├── observability/                    # 崩溃上报、日志、埋点客户端
│   │   └── permissions/                      # 相机、相册、定位等通用权限适配
│   └── test/                                 # 全局测试工具、Mock 和测试渲染器
│       ├── fixtures/
│       ├── mocks/
│       └── render.tsx
├── e2e/                                      # Detox 等端到端测试
├── scripts/                                  # 构建、发布、维护脚本
├── package.json
└── tsconfig.json
```

## 3. 目录职责与禁止事项

| 目录 | 放什么 | 不放什么 |
| --- | --- | --- |
| `app/` | 根组件、Provider、导航器、启动初始化、全局异常处理 | 具体业务规则、业务 API 调用、业务组件实现 |
| `app/navigation` | Navigator、路由类型、Deep Link、导航守卫 | `LoginScreen`、`ProjectDetailScreen` 的实现 |
| `features/<feature>` | 一个业务概念的全部代码，例如用户、项目、订单、账单 | 无业务含义的网络客户端；其他 Feature 的内部文件 |
| `presentation` | Screen、Feature 专属组件、视图 Hook、表单展示逻辑 | HTTP 请求细节、持久化实现、跨 Feature 基础 UI |
| `application` | 用例、状态机、Store、命令编排 | JSX UI、HTTP DTO、直接操作原生 API |
| `domain` | 实体、纯业务规则、Repository 接口 | React、React Native、Axios、Zustand、接口 DTO |
| `data` | API 客户端调用、DTO、Mapper、Repository 实现、缓存实现 | JSX、导航调用、业务展示状态 |
| `shared/ui` | 没有业务语义的可复用基础 UI | `ProjectStatusBadge`、`UserRoleSelector` 等业务组件 |
| `shared/components` | 两个及以上 Feature 稳定复用的组合组件 | 单个 Feature 或单个 Screen 的私有组件 |
| `core` | 通用技术 SDK、平台适配器、配置、安全存储、网络基础能力 | 业务 Service、业务类型、业务常量 |
| `assets` | 静态图片、字体、图标源文件 | 从接口下载的运行时资源、业务逻辑 |

## 4. Feature 内部调用关系

```text
Presentation → Application → Domain
                    ↓           ↑
                  Data ─────────┘
                    ↓
                  Core
```

| 调用方 | 允许调用 | 说明 |
| --- | --- | --- |
| `presentation` | `application`、`domain` 的展示类型、`shared` | Screen 通过用例或状态读取数据，不直接拼装请求。 |
| `application` | `domain`、`data`、`core` | 用例执行规则与流程；Store 只保存 UI 需要的客户端状态。 |
| `domain` | 仅自身 | 领域层必须保持平台无关、可脱离 React Native 单测。 |
| `data` | `domain`、`core` | Repository 实现满足领域定义的接口，并将 DTO 映射为领域对象。 |
| `app` | Feature 的 `index.ts`、`shared`、`core` | 导航层只导入 Feature 的公开 Screen 或入口。 |

## 5. 状态管理与数据请求

远程数据和本地交互状态不是同一种东西，必须分开管理。

| 状态类型 | 固定位置 | 实施规范 | 禁止方式 |
| --- | --- | --- | --- |
| 服务端状态 | `features/<feature>/data` 与 `presentation/hooks` | TanStack Query；Query Key 和请求函数按 Feature 管理 | 将接口响应整体复制到全局 Store |
| Screen 局部交互状态 | `features/<feature>/presentation` | 使用 `useState` | 创建 Zustand Store 或全局 Store |
| Feature 共享交互状态 | `features/<feature>/application/state` | 使用 Zustand Feature Store | 将 Store 放入顶层 `stores/` |
| 跨 Feature 的客户端状态 | 所属 Feature 的 `application/state`，由 `app/providers` 注册 | 使用 Zustand，并仅用于认证会话、主题、语言 | 创建无边界的全局状态目录 |
| 表单状态 | 对应 Feature 的 `presentation` | 使用 React Hook Form 与 Zod Schema | 在 Zustand 中保存未提交的表单值 |
| 持久化缓存 | `core/storage` 提供适配器，Feature 的 `data` 定义业务 Key 与读写时机 | 普通数据使用 MMKV；敏感数据使用 Keychain/Keystore | Feature 直接调用存储 SDK |

## 6. 导航架构

| 内容 | 放置位置 | 规则 |
| --- | --- | --- |
| 根导航器 | `app/navigation/RootNavigator.tsx` | 负责认证区与主应用区切换。 |
| 业务区域导航器 | `app/navigation/<Feature>Navigator.tsx` | 仅声明该区域路由及转场。 |
| Screen 实现 | `features/<feature>/presentation/screens` | Navigator 只导入 Screen，不承载 Screen 逻辑。 |
| 路由参数类型 | `app/navigation/types.ts` | 统一维护，禁止任意字符串路由名。 |
| Deep Link | `app/navigation/linking.ts` | URL 映射集中维护并有测试覆盖。 |

## 7. 原生平台与跨端规则

| 场景 | 放置位置 | 规则 |
| --- | --- | --- |
| iOS/Android 原生实现 | `ios/`、`android/` | 遵循原生工程约定，不混入 TypeScript 业务代码。 |
| 原生能力的 JS/TS 接口 | `core/native/` | 对业务层暴露稳定 TypeScript 接口，隐藏原生模块细节。 |
| 平台差异 UI | Feature 的 `presentation` 下的 `Component.ios.tsx`、`Component.android.tsx` | 仅在 UI 或行为存在平台差异时使用，公共逻辑放同名无平台后缀文件。 |
| 新原生模块 | `core/native/` + `ios/` + `android/` | 使用成熟社区库；自建模块必须定义错误与权限契约。 |
| 权限申请 | `core/permissions/` | Feature 声明业务意图，权限细节由 Core 统一处理。 |



## 8. 命名、公开 API 与依赖约束

| 对象 | 规则 | 示例 |
| --- | --- | --- |
| Feature 目录 | 单数、小写 kebab-case | `features/project`、`features/user-profile` |
| React 组件与 Screen | PascalCase | `ProjectDetailScreen.tsx`、`ProjectCard.tsx` |
| Hook | `use` 开头 | `useProjectDetail.ts` |
| 用例 | 动词开头 | `create-project.ts`、`sign-in.ts` |
| Repository 接口 | 名词加 `Repository` | `ProjectRepository.ts` |
| Repository 实现 | 接口名加 `Impl` | `ProjectRepositoryImpl.ts` |
| DTO | 用 `Dto` 后缀 | `ProjectDto.ts` |
| Feature 出口 | 每个 Feature 用 `index.ts` 显式导出允许跨模块使用的内容 | `export { ProjectDetailScreen } ...` |

| 依赖规则 | 要求 |
| --- | --- |
| Feature 间依赖 | 禁止导入另一 Feature 的内部路径；跨 Feature 协作只能导入对方 `index.ts` 明确公开的契约。 |
| Shared 与 Core | 不得导入 `app` 或 `features`，保证底层稳定。 |
| Domain | 不得导入 React、React Native、导航库、HTTP 库、状态库或原生 SDK。 |
| 导入别名 | 使用 `@/app`、`@/features`、`@/shared`、`@/core`，禁止跨目录深层相对路径。 |
| 依赖检查 | 使用 `eslint-plugin-boundaries` 自动阻止反向依赖和 Feature 深层导入。 |
