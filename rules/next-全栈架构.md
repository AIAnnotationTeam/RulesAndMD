# Next.js 中大型全栈项目目录规范

本规范适用于使用 TypeScript、App Router 的中大型 Next.js 全栈项目。项目可同时包含用户前台、管理后台、开放 API、Webhook 和定时任务。

目录以**路由边界**和**业务域**为主，以**运行环境**和**复用范围**为辅助边界：业务代码按功能内聚，框架入口保持轻薄，服务端数据访问集中且安全。

## 1. 目录归属原则

| 原则 | 规则 |
| --- | --- |
| 先判断业务归属 | 能明确归入一个业务域的代码，必须放入该业务域的 `features/<feature>`，不能先放入 `lib` 或 `components`。 |
| 再判断复用范围 | 仅在两个及以上独立业务域稳定复用，才进入共享目录。复用一次不等于通用。 |
| 路由不承载业务 | `app` 只处理路由、布局、页面组装、请求与响应适配；复杂规则、数据库读写不得放入其中。 |
| 服务端边界明确 | 数据库、密钥、鉴权、授权和第三方服务端 SDK 只能在服务端模块中使用。 |
| 第三方配置统一管理 | 第三方服务的环境配置必须统一从项目根目录的环境文件读取，经 `src/lib/env` 校验并导出；禁止在代码中硬编码配置值。 |
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
│   │   │   └── webhooks/stripe/route.ts # /api/webhooks/stripe
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
├── docs/                                # 架构与业务文档
├── .env.example                         # 环境变量名称、用途与占位值，提交版本控制
├── .env.local                           # 本地真实配置，禁止提交版本控制
├── .env.development.local               # 开发环境覆盖配置，禁止提交版本控制
├── .env.production.local                # 生产环境覆盖配置，由部署流程生成，禁止提交版本控制
├── .gitignore                           # 忽略真实环境文件，仅保留 .env.example
├── next.config.ts
├── tsconfig.json
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

第三方服务地址、应用 ID、API Key 等配置统一从项目根目录的 `.env.local` 等环境文件读取，经 `src/lib/env` 集中校验与导出，禁止写入代码。服务端密钥不得暴露给客户端，真实环境文件不得提交版本控制。
