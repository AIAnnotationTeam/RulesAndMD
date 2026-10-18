# Next.js 通用四层架构目录模板

当前为 TypeScript、App Router 的中大型 Next.js 项目模版，路由固定放在 `src/app`。

本模板结合按功能组织代码与单向依赖原则，将源码划分为四层。层级和业务目录名称属于团队架构约定，并非 Next.js 官方规定的四层标准；`app` 及路由特殊文件遵循 Next.js 框架约定。

## 1. 四层职责

| 层级 | 名称 | 目录 | 职责 |
| --- | --- | --- | --- |
| L0 | Application（应用层） | `src/app`、`src/proxy.ts` | 路由、应用启动、布局、页面组装、HTTP 协议适配 |
| L1 | Business（业务层） | `src/features` | 具体业务场景、业务 UI、交互、业务规则和数据访问 |
| L2 | Business Architecture（业务能力层） | `src/capabilities` | 跨业务复用、标准化的账户、组织、权限等业务能力 |
| L3 | Infrastructure（基础设施层） | `src/shared` | 与具体业务无关的 UI、网络、数据库连接、日志及通用工具 |

四层按业务职责划分，服务端和客户端按运行环境划分。L1、L2 都可以包含服务端与客户端代码；`server` 不单独构成一个架构层。

## 2. 通用目录结构

以下使用 `resource` 代表具体业务模块，新项目中替换为实际业务名称。可选目录按需创建，不要求每个模块都建立全部子目录。

```text
project/
├── public/                              # 可通过 URL 访问的静态资源
│   ├── images/
│   ├── icons/
│   └── fonts/
├── src/                                 # 应用源码目录
│   ├── app/                             # L0：路由与应用组装
│   │   ├── layout.tsx                   # 根布局
│   │   ├── page.tsx                     # 首页 /
│   │   ├── globals.css                  # 全局样式入口
│   │   ├── providers.tsx                # 客户端 Provider 组装，自定义文件名
│   │   ├── loading.tsx                  # 路由加载界面
│   │   ├── error.tsx                    # 错误边界，Client Component
│   │   ├── global-error.tsx             # 全局错误界面
│   │   ├── not-found.tsx                # 未找到页面
│   │   ├── favicon.ico
│   │   ├── robots.ts                    # 可选 SEO 元数据路由
│   │   ├── sitemap.ts
│   │   ├── (auth)/                      # 路由组，不计入 URL
│   │   │   ├── layout.tsx
│   │   │   ├── login/
│   │   │   │   └── page.tsx             # /login，组装登录业务 UI
│   │   │   └── register/
│   │   │       └── page.tsx             # /register
│   │   ├── (workspace)/
│   │   │   ├── layout.tsx               # 工作区布局与模块组装
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx             # /dashboard
│   │   │   └── resources/
│   │   │       ├── page.tsx             # /resources
│   │   │       ├── _components/         # 可选，仅路由组装使用的组件
│   │   │       │   └── resource-page-shell.tsx
│   │   │       └── [id]/
│   │   │           ├── page.tsx         # /resources/:id
│   │   │           ├── loading.tsx
│   │   │           └── error.tsx
│   │   ├── (management)/
│   │   │   └── admin/                   # 实际 URL 段
│   │   │       ├── layout.tsx
│   │   │       ├── page.tsx             # /admin
│   │   │       └── users/
│   │   │           └── page.tsx         # /admin/users
│   │   └── api/                         # HTTP 入口，调用业务或能力模块
│   │       ├── resources/
│   │       │   ├── route.ts             # GET/POST /api/resources
│   │       │   └── [id]/
│   │       │       └── route.ts         # GET/PATCH/DELETE /api/resources/:id
│   │       └── webhooks/
│   │           └── provider/
│   │               └── route.ts        # 第三方回调入口
│   ├── features/                        # L1：具体业务模块
│   │   ├── resource/
│   │   │   ├── components/              # 业务列表、详情、编辑表单
│   │   │   ├── hooks/                   # 当前业务的客户端 Hook
│   │   │   ├── api/                     # 浏览器请求函数，不是 HTTP 路由
│   │   │   ├── stores/                  # 可选，模块内共享的客户端状态
│   │   │   ├── server/                  # 当前业务的服务端实现
│   │   │   │   ├── resource.service.ts       # 业务用例、规则与流程
│   │   │   │   ├── resource.queries.ts       # 授权后的读取入口，返回最小 DTO
│   │   │   │   ├── resource.repository.ts    # 具体业务的数据库操作
│   │   │   │   ├── resource.actions.ts       # 可选，Server Actions 入口
│   │   │   │   └── resource.mapper.ts        # 持久化模型转 DTO
│   │   │   ├── schemas.ts               # 业务输入校验与共享结构
│   │   │   ├── types.ts                 # 当前模块的数据契约
│   │   │   ├── constants.ts             # 当前模块的业务常量
│   │   │   └── utils.ts                 # 当前模块的纯函数
│   │   ├── authentication/              # 登录、注册等具体业务流程
│   │   │   ├── components/
│   │   │   ├── hooks/
│   │   │   ├── server/
│   │   │   └── schemas.ts
│   │   └── user-management/             # 用户管理业务
│   │       ├── components/
│   │       ├── hooks/
│   │       ├── server/
│   │       └── types.ts
│   ├── capabilities/                    # L2：跨业务复用的标准化业务能力
│   │   ├── account/
│   │   │   ├── components/              # 可选，标准化账户相关组件
│   │   │   ├── hooks/                   # 可选，账户状态读取
│   │   │   ├── server/
│   │   │   │   ├── session.ts           # 身份验证、会话管理
│   │   │   │   └── account.repository.ts
│   │   │   ├── schemas.ts
│   │   │   └── types.ts                 # 账户能力对外契约
│   │   ├── organization/
│   │   │   ├── server/                  # 组织上下文、成员关系查询
│   │   │   └── types.ts
│   │   └── access-control/
│   │       ├── server/                  # 通用权限判定，接收身份与资源上下文
│   │       └── types.ts
│   ├── shared/                          # L3：与具体业务无关的基础设施
│   │   ├── ui/                          # Button、Dialog、Input 等基础 UI
│   │   ├── hooks/                       # use-media-query 等通用 Hook
│   │   ├── http/
│   │   │   ├── client.ts                # 浏览器 HTTP 请求封装
│   │   │   └── server.ts                # 服务端 HTTP 请求封装
│   │   ├── database/
│   │   │   └── server/
│   │   │       └── client.ts            # 数据库连接、底层驱动初始化
│   │   ├── storage/
│   │   │   └── server/
│   │   │       └── client.ts            # 对象存储底层客户端
│   │   ├── email/
│   │   │   └── server/
│   │   │       └── client.ts            # 通用邮件发送客户端
│   │   ├── logging/                     # 日志工具
│   │   ├── config/
│   │   │   ├── env.client.ts            # 可公开的客户端配置
│   │   │   └── env.server.ts            # 服务端环境变量校验
│   │   ├── styles/                     # 主题变量、基础样式
│   │   ├── types/                      # 与业务无关的基础类型
│   │   └── utils/                      # 日期、格式化等通用纯函数
│   └── proxy.ts                         # L0：可选，Next.js 16 请求预处理
├── tests/
│   ├── e2e/                             # 关键端到端流程
│   └── integration/                     # 接口、数据访问与模块集成验证
├── docs/                                # 架构及业务说明
├── scripts/                             # 数据迁移、维护脚本
├── .env.example                         # 配置示例，不含真实密钥
├── .env.local                           # 本地配置，不提交 Git
├── next.config.ts
├── next-env.d.ts                        # Next.js 生成的类型声明
├── tsconfig.json
├── eslint.config.mjs
├── postcss.config.mjs
├── package.json
└── package-lock.json                    # 使用对应包管理器的锁文件
```

## 3. 依赖规则

箭头表示源码导入方向，允许跨层使用，无需逐层增加透传封装。

```text
L0 Application  → L1、L2、L3
L1 Business     → L2、L3
L2 Capabilities → L3
L3 Shared       → 不依赖上层
```

- **L0 不包含业务规则。** 页面和接口处理框架参数、请求解析、模块组装及响应映射，业务判断交给 L1 或 L2。
- **L1 不依赖 L0。** 不同业务模块之间禁止横向强依赖。通过 L0 组合独立模块；可复用的业务能力下沉到 L2，具体业务编排仍留在 L1。
- **L2 不依赖 L0、L1。** L2 模块之间禁止横向强依赖；通过显式参数、上下文或回调传入协作所需数据，避免账户、组织、权限能力相互导入。
- **L3 不依赖 L0、L1、L2。** L3 内模块可横向依赖，但应避免循环依赖。
- **共享契约归所属层。** 具体业务类型留在 L1，能力契约留在 L2；不能为了复用而将业务类型全部放入 L3。
- **按职责判断归属。** 数据库连接属于 L3，包含具体资源字段与查询语义的 Repository 属于 L1 或 L2。第三方适配中涉及具体业务规则的部分也应留在上层。
- **应用配置归应用层。** 导航、菜单和业务模块注册放在 L0；通用环境变量读取与技术配置放在 L3。
- 可通过 ESLint 导入限制等方式检查反向依赖及跨模块直接导入。

## 4. Next.js 运行环境与数据访问约定

### 4.1 服务端与客户端边界

- 页面和布局默认使用 Server Components；交互状态、事件、浏览器 API 所在边界使用 `'use client'`。
- `server/` 只是目录命名。服务端专用入口和敏感实现使用 `import 'server-only'`，防止客户端误导入。
- Server Actions 入口使用 `'use server'`，放在所属业务或能力模块中；避免通过同一个聚合出口混合导出客户端代码与服务端代码。
- 服务端返回给客户端的数据使用安全、最小化的 DTO；密钥不得通过 props、接口响应或公开环境变量泄露。

### 4.2 典型调用路径

```text
读取：
L0 Server Component → L1 授权查询入口 → L1 Repository → L3 数据库客户端

写入：
Client Component → Route Handler / Server Action → L1 业务服务
                → L2 身份与权限能力 → L1 Repository → L3 数据库客户端

共享能力：
L1 业务模块 → L2 标准化业务能力 → L3 基础设施
```

上述路径中的授权必须在受保护的数据读写前完成；箭头表示调用关系示意，不限定方法拆分数量。

- 全栈项目在 L1、L2 内形成专门的服务端数据访问层（DAL），统一授权检查和 DTO 输出。DAL 是职责边界，不额外增加第五层。
- Server Components 直接调用服务端查询入口，无需经由本站 `/api` 增加 HTTP 往返；已有独立后端时，可由服务端模块访问该后端。
- Route Handlers 与 Server Actions 按调用需求选择，不要求同一个操作实现两套入口。Server Actions 主要用于数据变更。
- 登录验证、角色与资源归属校验必须落实到服务端业务或数据入口；不能只依赖客户端跳转、布局或 Proxy。

### 4.3 框架目录约定

- 路由固定放在 `src/app`；`public`、配置文件和环境变量文件放在项目根目录。
- `(group)` 路由组不进入 URL；`_components` 等私有目录不参与路由。
- `app` 内专属组件只承载路由组装；包含业务规则和交互的组件放在对应 `features` 模块。
- `route.ts` 可位于 `app` 下其他路由段，`app/api` 是本模板采用的组织方式；同一路由段不能同时定义 `page.tsx` 与 `route.ts`。
- Next.js 16 使用 `src/proxy.ts` 作为本模板的请求预处理入口；旧版本可能使用 `middleware.ts`。仅有预处理需求时创建。


