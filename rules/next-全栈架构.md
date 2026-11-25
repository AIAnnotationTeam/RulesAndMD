# Next.js 中大型全栈项目目录规范

## 外部平台能力选型（强制）

先执行[统一选型规则](平台能力选择原则.md#外部平台能力选型强制)，读取 [Cloudflare 能力清单](../平台能力/Cloudflare能力清单.md) 与 [百炼能力清单](../平台能力/百炼平台能力清单.md)。框架和目录规范不预先绑定云服务供应商。

| 能力 | 新项目优先路线 | 实施要求 |
| --- | --- | --- |
| 部署与后端 | Workers + 与当前 Next.js 版本兼容的框架适配路径 | 先验证运行时和依赖；需要系统二进制与完整进程时采用 Containers，不把 Node.js 兼容理解为完整 Node 运行环境 |
| 认证与数据 | 产品账号评估 Better Auth + D1，受控入口评估 Access；数据库独立验证 D1 | 认证、数据库与业务授权分开确定；保留 PostgreSQL / Supabase 的项目记录特性与迁移依据，不默认安装全套 Supabase SDK |
| 文件与任务 | R2；异步处理按语义选择 Queues、Workflows、Cron | R2 binding 与 S3 API 按执行位置确定；不为使用所有产品同时引入多套队列 |
| 实时 | 房间状态优先 DO，音视频优先 Realtime SFU / TURN / 媒体适配器 | 模型长连接网关单独验证，运行时确定后才选传输库；独立 Node Gateway 不是所有项目的默认单元 |
| AI | 百炼清单按能力确定模型，Cloudflare Workers AI 按适用性评估 | 对话、同传、ASR、TTS、图像及视频模型分别配置；业务不直接依赖供应商事件格式 |
| 监控与通知 | Issues、Logs、Traces、Alerts；事务邮件使用 Email Sending，Web 通知评估 Web Push | 校验套餐、域名、投递与平台边界，按需使用 Analytics Engine / Zaraz |
| 支付 | 国内普通 Web 支付优先支付宝，海外评估 Stripe | 按清单支付场景表固定选型；x402 / MPP、爬虫收费、模型成本支付独立判断，不作为普通收银默认依赖 |

已确认项目方案优先于通用模板；上述优先路线需完成适用性验证后形成唯一实施方案。现有工程不因模板更新自动迁移数据库、认证或网关。

本规范适用于使用 TypeScript、App Router 的中大型 Next.js 全栈项目。项目可同时包含用户前台、管理后台、开放 API、Webhook 和定时任务。

目录以**路由边界**和**业务域**为主，以**运行环境**和**复用范围**为辅助边界：业务代码按功能内聚，框架入口保持轻薄，服务端数据访问集中且安全。

## 1. 目录归属原则

| 原则 | 规则 |
| --- | --- |
| 先判断业务归属 | 能明确归入一个业务域的代码，必须放入该业务域的 `features/<feature>`，不能先放入 `lib` 或 `components`。 |
| 再判断复用范围 | 仅在两个及以上独立业务域稳定复用，才进入共享目录。复用一次不等于通用。 |
| 路由不承载业务 | `app` 只处理路由、布局、页面组装、请求与响应适配；复杂规则、数据库读写不得放入其中。 |
| 服务端边界明确 | 数据库、密钥、鉴权、授权和第三方服务端 SDK 只能在服务端模块中使用。 |
| 第三方配置统一管理 | 第三方标量配置本地从项目环境文件读取，生产由部署平台注入，经 `src/lib/env` 校验导出；资源绑定通过运行时适配器获取，禁止硬编码真实配置值。 |
| 先就近后上提 | 代码默认放在最小有效作用域；确认跨域稳定复用后才上提，避免过早抽象。 |

## 2. 标准目录模板

```text
project/
├── public/                              # 可公开访问的静态资源
│   ├── images/
│   ├── icons/
│   └── fonts/
├── src/
│   ├── app/                             # Next.js 路由与框架文件
│   │   ├── (site)/                      # 用户前台路由组，不进入 URL
│   │   │   ├── layout.tsx               # 前台布局
│   │   │   ├── page.tsx                 # /
│   │   │   ├── pricing/page.tsx         # /pricing
│   │   │   └── workspace/
│   │   │       ├── layout.tsx           # 登录后的用户工作区布局
│   │   │       └── projects/
│   │   │           ├── page.tsx         # /workspace/projects
│   │   │           └── [projectId]/
│   │   │               └── page.tsx     # /workspace/projects/:projectId
│   │   ├── (auth)/                      # 身份认证路由组
│   │   │   ├── layout.tsx
│   │   │   ├── login/page.tsx           # /login
│   │   │   └── register/page.tsx        # /register
│   │   ├── (admin)/                     # 管理后台路由组
│   │   │   └── admin/
│   │   │       ├── layout.tsx           # /admin 的后台布局与入口权限检查
│   │   │       ├── page.tsx             # /admin
│   │   │       └── users/
│   │   │           └── page.tsx         # /admin/users
│   │   ├── api/                         # 对外 HTTP API 与第三方回调
│   │   │   ├── v1/projects/route.ts     # /api/v1/projects
│   │   │   └── webhooks/                  # 仅创建已启用渠道的 alipay、stripe 等回调
│   │   ├── layout.tsx                   # 根布局
│   │   ├── error.tsx
│   │   ├── global-error.tsx
│   │   ├── not-found.tsx
│   │   ├── globals.css
│   │   ├── robots.ts
│   │   └── sitemap.ts
│   ├── features/                        # 业务域，项目的主要代码归属地
│   │   ├── project/
│   │   │   ├── components/              # 项目业务专属 UI
│   │   │   ├── hooks/                   # 项目业务专属客户端 Hook
│   │   │   ├── server/                  # 项目的 DAL、领域服务与持久化实现
│   │   │   │   ├── project.dal.ts
│   │   │   │   ├── project.service.ts
│   │   │   │   └── project.repository.ts
│   │   │   ├── actions.ts               # 项目业务的 Server Actions
│   │   │   ├── schema.ts                # 输入校验 Schema
│   │   │   ├── types.ts                 # 业务类型与 DTO
│   │   │   ├── constants.ts
│   │   │   └── utils.ts
│   │   ├── user/
│   │   ├── organization/
│   │   ├── billing/
│   │   └── authentication/
│   ├── components/                      # 跨业务域复用的 UI
│   │   ├── ui/                          # Button、Dialog、Input 等基础组件
│   │   └── shared/                      # AppHeader、DataTable 等组合通用组件
│   ├── hooks/                           # 跨业务域复用的客户端 Hook
│   ├── lib/                             # 无业务归属的技术能力
│   │   ├── auth/                        # 会话解析、当前用户获取、认证 SDK 配置
│   │   ├── db/                          # 数据库客户端与 ORM 初始化
│   │   ├── env/                         # 第三方环境配置唯一读取入口，分离服务端与客户端变量
│   │   │   ├── server.ts                # 服务端变量校验与导出，添加 server-only
│   │   │   └── client.ts                # 仅校验并导出 NEXT_PUBLIC_ 公开变量
│   │   ├── http/                        # 通用 HTTP 客户端、响应工具
│   │   ├── storage/                     # 对象存储客户端
│   │   ├── observability/               # 日志、监控、埋点客户端
│   │   └── utils/                       # 日期、字符串等无业务纯函数
│   ├── styles/                          # 主题变量与非路由专属样式
│   └── proxy.ts                         # 可选：Next.js 16 请求预处理入口
├── tests/
│   ├── e2e/                             # 关键用户流程
│   ├── integration/                     # Route Handler、DAL 与外部服务集成
│   └── unit/                            # 纯函数和业务规则
├── scripts/                             # 维护、导入、迁移脚本
├── migrations/                          # 使用 D1 时保存迁移；其他数据库按已确定工具归档
├── docs/                                # 架构与业务文档
├── .env.example                         # 沿用通用变量模板命名与平台分组，仅保留占位值
├── .env.local                           # 本地真实配置，禁止提交版本控制
├── .env.development.local               # 开发环境覆盖配置，禁止提交版本控制
├── .env.production.local                # 生产环境覆盖配置，由部署流程生成，禁止提交版本控制
├── .gitignore                           # 忽略真实环境文件，仅保留 .env.example
├── next.config.ts
├── tsconfig.json
├── wrangler.jsonc                       # 采用 Workers 时的部署、资源绑定与兼容配置
└── package.json
```

## 3. 各目录职责与禁止事项

| 目录 | 放什么 | 不放什么 |
| --- | --- | --- |
| `app/` | `page.tsx`、`layout.tsx`、`loading.tsx`、`error.tsx`、`route.ts`、路由级组装代码 | 复杂业务规则、直接数据库查询、可复用业务组件 |
| `app/(site)` | 用户前台页面和布局 | 管理后台页面；业务逻辑实现 |
| `app/(admin)` | 管理后台页面、布局和后台入口权限检查 | 仅前台使用的页面；用户管理的具体读写逻辑 |
| `app/api` | 外部 HTTP 协议适配、Webhook 验签、调用对应业务服务 | 给 Server Component 自己调用的内部接口；核心业务规则 |
| `features/<feature>` | 一个业务域的 UI、类型、校验、交互、服务端实现 | 无业务归属的技术客户端；其他业务域的内部实现 |
| `features/<feature>/components` | 仅该业务域使用的组件 | 通用基础 UI；只属于单一路由且不会复用的极小组件 |
| `features/<feature>/server` | DAL、授权后的查询、业务服务、Repository、服务端第三方适配 | Client Component、浏览器 API、跨业务域的基础数据库客户端 |
| `features/<feature>/actions.ts` | 该业务域的薄 Server Action 入口 | 未授权的数据写入；与该业务无关的 Action |
| `components/ui` | 无业务含义的基础 UI 组件 | `UserRoleSelector`、`ProjectCard` 等业务组件 |
| `components/shared` | 两个及以上业务域稳定复用的组合组件 | 单次复用或包含特定业务规则的组件 |
| `hooks/` | 跨业务域的客户端 Hook | `useProjectMembers` 等业务域专属 Hook |
| `lib/` | 无业务归属的技术配置、客户端、纯工具函数 | 业务 Service、业务 DTO、领域常量；不可分类的杂项 |
| `lib/env/` | 从环境文件加载后的环境变量中读取、校验并按服务端/客户端导出配置 | 硬编码真实配置值；将服务端密钥导出给客户端；业务文件自行读取环境变量 |
| `styles/` | 全局主题、设计 Token、基础样式 | 单组件样式逻辑；业务规则 |

## 4. 路由与业务域的协作

路由层只选择和组装业务模块，同一个业务域可被用户前台、管理后台、Route Handler 和 Server Action 共同调用。

```text
app/(site)/workspace/projects/page.tsx ─┐
app/(admin)/admin/projects/page.tsx     ├─> features/project
app/api/v1/projects/route.ts            │   ├─ components/
features/project/actions.ts             │   └─ server/
                                         └─> lib/db、lib/auth
```

| 场景 | 放置位置 | 调用目标 |
| --- | --- | --- |
| 前台项目列表页面 | `app/(site)/workspace/projects/page.tsx` | `features/project/server/project.dal.ts` 与项目组件 |
| 后台项目审核页面 | `app/(admin)/admin/projects/page.tsx` | `features/project/server/project.dal.ts` 与项目组件 |
| 创建项目表单 | `features/project/components/` | `features/project/actions.ts` |
| 创建项目的业务规则 | `features/project/server/project.service.ts` | Repository、认证与授权能力 |
| `POST /api/v1/projects` | `app/api/v1/projects/route.ts` | `project.service.ts` |
| 数据库连接 | `lib/db/` | 仅由服务端 DAL 或 Repository 导入 |

## 5. 服务端数据访问规则

### 5.1 分层职责

| 模块 | 职责 | 必须遵守 |
| --- | --- | --- |
| Route Handler | 解析 HTTP 请求、校验协议、映射 HTTP 响应 | 不实现核心业务规则；调用 Service 或 DAL |
| Server Action | 处理表单或客户端发起的变更请求 | Action 内重新认证和授权；保持薄入口 |
| DAL | 授权后的读取和安全 DTO 输出 | 添加 `import 'server-only'`；不返回超出调用方需要的字段 |
| Service | 执行业务流程、写入规则、事务编排 | 不依赖 `app`；不使用浏览器 API |
| Repository | 特定业务实体的持久化查询 | 不处理 HTTP；不替代授权判断 |
| `lib/db` | ORM 或数据库连接初始化 | 不包含任何实体字段和业务查询语义 |

### 5.2 调用路径

```text
读取：
Server Component → feature DAL → Repository → lib/db

写入：
Client Component → Server Action 或 Route Handler → Service → Repository → lib/db
                                                        └→ 授权检查
```

| 规则 | 要求 |
| --- | --- |
| Server Component 读取数据 | 直接调用 DAL；不得通过本站 `/api` 再发一次 HTTP 请求。 |
| 数据修改 | 使用 Server Action 或 Route Handler；两者都必须执行认证、授权和输入校验。 |
| 服务端专属文件 | 在 DAL、Service、Repository 中添加 `import 'server-only'`。 |
| 返回客户端的数据 | 使用最小 DTO；不得将 ORM 实体、密钥或内部字段直接传给客户端。 |
| 页面权限检查 | 可改善体验，但不能替代 Action、Route Handler 和 DAL 中的授权检查。 |


## 6. 命名与依赖规则

| 对象 | 命名规则 | 示例 |
| --- | --- | --- |
| 业务目录 | 单数、小写 kebab-case | `features/project`、`features/user-profile` |
| React 组件 | PascalCase 文件名 | `ProjectForm.tsx` |
| Hook | `use-` 前缀的 kebab-case 文件名 | `use-project-filters.ts` |
| DAL | `<feature>.dal.ts` | `project.dal.ts` |
| Service | `<feature>.service.ts` | `project.service.ts` |
| Repository | `<feature>.repository.ts` | `project.repository.ts` |
| 路由私有目录 | `_` 前缀 | `_components`、`_lib` |

| 导入方向 | 允许 | 禁止 |
| --- | --- | --- |
| `app` | 导入 `features`、`components`、`hooks`、`lib` | 被 `features` 或 `lib` 反向导入 |
| `features/<a>` | 导入自身目录、`components`、`hooks`、`lib` | 直接导入 `features/<b>` 的内部实现；导入 `app` |
| `components`、`hooks`、`lib` | 导入更基础的共享代码 | 导入 `app` 或具体 `features` |
| 一个业务域需要另一业务域能力 | 通过公开的服务契约、参数或上层编排协作 | 横向引用另一个业务域的内部 `server/`、`components/` |

## 7. 路由目录补充规则

| 目录或文件 | 用途 |
| --- | --- |
| `(group)` | 组织路由但不影响 URL，例如 `(site)`、`(admin)`、`(auth)`。 |
| `_folder` | 路由段的私有实现细节，不参与路由，例如 `app/(site)/pricing/_components`。 |
| `page.tsx` | 页面入口，负责读取参数、调用业务模块、渲染页面。 |
| `layout.tsx` | 持久布局与路由区域级组装；可做入口级会话检查。 |
| `loading.tsx`、`error.tsx`、`not-found.tsx` | Next.js 路由状态文件。 |
| `route.ts` | 外部 HTTP 接口或 Webhook；同一路由段不得与 `page.tsx` 并存。 |

## 8. 第三方环境变量管理（强制）

新项目以 [Next 通用环境变量模板](../Next全局变量/env.example) 为命名契约：使用相同平台能力时沿用已有变量名，按平台类型分组；模板中未覆盖的配置按其命名规则补充，同步维护模板与项目 `.env.example`。仅启用实际使用的配置组，不填写或校验未使用的平台。

本地标量配置从应用根目录的 `.env.local` 等环境文件读取，线上配置与 Secrets 由部署平台注入，经 `src/lib/env` 集中校验与导出。项目 `.env.example` 仅保留名称、用途和占位值；真实配置禁止硬编码或提交版本控制，服务端密钥不得暴露给客户端。

采用 Workers 时，线上标量配置与 Secrets 由部署平台注入，D1、R2、KV、DO 等原生绑定经框架适配器提供的运行时上下文获取，不序列化进 `.env.local`。Wrangler 本地调试按其加载机制使用 `.dev.vars` 等本地文件，禁止提交真实值；`src/lib/env` 统一导出校验后的标量配置，独立运行时适配器提供类型化资源绑定。只有所选业务启用时才校验对应配置。

## 9. 默认第三方库基线（强制纳入架构模板）

以下库作为中大型 Next.js 全栈项目的默认技术基线，目录、依赖方向和环境配置必须按本节执行。业务功能库仍需在启用对应功能时安装，禁止为了凑齐清单把无关库加入运行依赖。

### 9.1 默认基础依赖

| 能力 | 默认第三方库 | 固定用途 | 归属位置 |
| --- | --- | --- | --- |
| Web 框架 | `next`、`react`、`react-dom` | App Router、页面渲染、服务端组件和客户端组件 | 项目根目录 `package.json` 声明依赖，路由位于 `src/app` |
| 样式与 CSS 构建 | `tailwindcss`、`postcss`、`@tailwindcss/postcss` | Tailwind 样式、全局主题、设计令牌和响应式布局 | `src/app/globals.css`、`src/styles` |
| 表单与运行时校验 | `react-hook-form`、`zod` | 表单状态、API 参数、Server Action、环境变量和第三方响应校验 | Feature `schema.ts`、`src/lib/env` |
| HTTP 客户端 | `axios` | 服务端第三方 API、Webhook 和需要拦截器的 HTTP 调用 | `src/lib/http`、服务端 Adapter |
| 客户端数据缓存 | `swr` | 客户端列表、详情、用户状态读取和缓存失效 | Feature Hooks |
| 图标与基础 UI | `lucide-react` | 统一图标组件，禁止页面直接混用图标来源 | `src/components/ui` |
| 日期时间 | `date-fns`、`date-fns-tz` | 日期格式化、时区转换和预约时间处理 | `src/lib/utils`、业务 Service |

### 9.2 默认开发、质量与测试依赖

| 能力 | 默认第三方库 | 固定用途 |
| --- | --- | --- |
| TypeScript 类型 | `typescript`、`@types/node`、`@types/react`、`@types/react-dom` | 类型检查和 Node/React 类型声明 |
| 代码检查 | `eslint`、`eslint-config-next`、`prettier` | ESLint 规则和统一代码格式 |
| 单元与组件测试 | `vitest`、`@testing-library/react`、`@testing-library/jest-dom`、`jsdom` | Domain、Service、Hook、表单和组件主流程测试 |
| 浏览器 E2E | `@playwright/test` | 登录、核心业务流程、权限和浏览器交互测试 |

### 9.3 按功能启用的预置库

| 功能 | 第三方库 | 启用后固定归属 |
| --- | --- | --- |
| 产品认证 | 采用 Better Auth 时安装 `better-auth` 及已验证的必要适配依赖 | `src/lib/auth` 初始化，认证 Feature 处理业务；不默认叠加 Supabase Auth |
| 保留 Supabase | `@supabase/supabase-js`、`@supabase/ssr` | 只有所选项目仍使用其能力时安装，分别放 `src/lib/db`、`src/lib/auth`；记录选型原因 |
| D1 数据访问 | Workers D1 binding；ORM 依赖由项目验证后确定 | `src/lib/db` 封装绑定访问，查询归 Feature Repository；不把本地 SQLite 或 PostgreSQL 测试引擎当成 D1 完整替身 |
| 独立 Node.js 网关 | `ws`、`@types/ws` | 仅明确选择 Node 运行单元时安装于对应 Gateway；Workers / DO 使用其运行时接口，不安装 `ws` 充当服务端 |
| 对象存储 | R2 binding；使用 S3 API / 签名时安装 `@aws-sdk/client-s3`、`@aws-sdk/s3-request-presigner` | `src/lib/storage`；按访问方式引入，SDK 命名不代表选用 AWS 存储，长期密钥仅在服务端 |
| 支付 | 国内支付宝使用 `alipay-sdk`；海外 Stripe 使用 `stripe` | 仅安装已启用渠道的 SDK，验证运行时兼容；`features/billing/server` 封装，Webhook 必须验签 |
| 文件解析 | `unpdf`、`mammoth` | 对应 Feature 的 `server` 和 Adapter |
| 浏览器录制 | `recordrtc`、`@types/recordrtc` | 对应媒体 Feature；必须先定义授权、存储和清理策略 |
| 二维码 | `qrcode`、`@types/qrcode` | 对应邀请或支付 Feature；不放入全局组件 |
| 音频编码 | `@breezystack/lamejs` | 音频 Feature；通过 Web Worker 执行编码 |
| PostgreSQL 集成测试 | `@electric-sql/pglite` | 仅 PostgreSQL 路线使用，位于 `tests/integration`；D1 路线使用对应本地运行时和隔离平台验证 |

### 9.4 依赖使用规则

| 规则 | 固定要求 |
| --- | --- |
| 依赖清单 | 新项目创建时先建立上述默认基础依赖和质量依赖；按功能启用预置库时同步更新 `package.json` 和架构文档 |
| 原生能力优先 | Next.js 原生 `fetch`、浏览器 Web API 或已有内部组件已经满足需求时，不重复引入第三方库 |
| 适配层 | 支付、对象存储、翻译、RTC、文件解析等第三方 SDK 必须放在 Adapter 中，业务层只依赖接口 |
| 环境变量 | 标量配置本地使用环境文件，生产使用平台配置与 Secrets；D1、R2 等资源绑定按第 8 节从运行时获取；客户端不得读取服务端密钥 |
| 测试门禁 | 每个新增业务功能至少覆盖一个主流程测试；第三方 SDK 使用 Mock 或沙箱，不把真实密钥写入测试 |
