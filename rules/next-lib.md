

# Next 全栈项目的常用第三方库

## 外部平台能力选型（强制）

按[统一选型规则](平台能力选择原则.md#外部平台能力选型强制)、[Cloudflare 能力清单](../平台能力/Cloudflare能力清单.md)与[百炼能力清单](../平台能力/百炼平台能力清单.md)先选平台再选 SDK；本地库与框架基线保持职责不变。云平台绑定、运行时接口不冒充第三方 npm 包，未启用的能力不安装依赖。 配置命名和平台分组沿用 [Next 通用环境变量模板](../Next全局变量/env.example)，读取、校验和 Workers 绑定规则遵守 [Next 架构规范](next-全栈架构.md#8-第三方环境变量管理强制)。


# 应用框架与界面

| 第三方库 | 简要介绍 |
| --- | --- |
| `next` | 基于 React 的全栈框架，提供页面路由、服务端渲染、HTTP API 和项目构建能力。 |
| `react` | 组件化界面库，负责页面展示、用户交互和状态管理。 |
| `react-dom` | 将 React 组件渲染到浏览器页面，并提供服务端渲染能力。 |
| `lucide-react` | React 图标组件库，提供统一的 SVG 图标。 |


# 网络通信工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `axios` | 基于 Promise 的 HTTP 客户端，支持浏览器和 Node.js，用于调用 HTTP API，提供请求与响应拦截器、超时配置和请求取消等能力。 |
| `ws` | 仅明确采用独立 Node.js 网关时安装；Workers / DO 使用自身运行时 WebSocket API，浏览器使用 WebSocket，不无条件创建网关。 |

# 数据缓存工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `swr` | React 数据请求与缓存库，支持缓存复用、请求去重、重新校验和写入后更新。可搭配原生 fetch 或 Axios，用于列表、详情等客户端数据读取。 |


# 音频编码工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `@breezystack/lamejs` | JavaScript MP3 编码库，可将 PCM 音频编码为 MP3。项目用于将包含候选人与 AI 双方声音的完整混音录音转换为 MP3，可配合 Web Worker 避免阻塞页面交互。 |

# 二维码工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `qrcode` | 二维码生成库，支持生成 Canvas、图片数据 URL 等形式的二维码。项目用于将支付链接展示为可扫描的支付二维码。 |


# 数据库集成测试工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `@electric-sql/pglite` | 仅保留 PostgreSQL 选型时用于迁移、约束和事务等测试，不代替生产数据库；D1 路线使用对应本地运行时与隔离平台验证，不用 PGlite 证明 D1 兼容。 |

# 代码格式工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `prettier` | 代码格式化工具，统一缩进、换行和排版。配合 ESLint 使用，分别负责代码格式与代码规则检查。 |





# 身份认证与数据库工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `better-auth` | 产品账号优先评估 Workers + D1 + Better Auth，验证后按需安装；服务端封装用户会话，邮件接入 Email Sending，业务权限独立实现。 |
| D1 binding（平台能力，非 npm 库） | 新项目云数据库优先评估，`lib/db` 适配、Repository 查询；ORM 由项目按需求验证后固定。 |
| Access（平台能力，非 npm 库） | 受控入口优先评估；验证码、Passkey 和身份源按清单选择，不自动代替产品权益与资源授权。 |
| `@supabase/supabase-js` | 仅已有工程复用或清单路线无法满足需求、已记录采用依据时安装；不作为所有新项目基础依赖。 |
| `@supabase/ssr` | 仅使用 Supabase 会话与 SSR 时安装，不与另一套默认认证重复初始化。 |


# 对象存储工具
| 第三方库 | 简要介绍 |
| --- | --- |
| R2 binding（平台能力，非 npm 库） | Workers 内部文件读写优先使用原生绑定；文件归属和权限由应用校验，不把绑定对象放入环境字符串。 |
| `@aws-sdk/client-s3` | 需要 S3 API 或签名流程时用于访问 R2；按执行环境确定访问方式，不同时为相同调用维护重复入口。 |
| `@aws-sdk/s3-request-presigner` | 生成带有效期的上传、下载签名 URL，让浏览器安全访问私有存储文件。 |


# 支付工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `alipay-sdk` | 国内普通 Web 支付优先，按签约产品接入，服务端签名与验签；验证 Workers 兼容性后采用，不默认支持自动扣款。 |
| `creem` | 海外普通 Web 收款采用 Creem，通过 TypeScript SDK 创建收银会话并处理订阅；完成商户审核，按原始请求体验签并幂等更新权益。 |
| `@creem_io/nextjs` | 采用官方 Next.js 路由与组件适配时按需安装；与 SDK 统一封装在一个支付 Adapter 内，验证部署运行时兼容性，不因存在支付业务就安装所有适配器。 |

机器付费 API / MCP、爬虫收费、模型成本支付分别按 Cloudflare 清单选择 x402 / MPP、Pay Per Crawl、AI Gateway Machine Payments；不是普通收银基线。

# 服务端 AI、实时媒体与可观测性

| 平台能力 / 接入工具 | 使用要求 |
| --- | --- |
| 百炼 API / 对应 SDK | 按百炼清单确定所需模型与协议；长期密钥保存在服务端，客户端不能直接配置模型管理凭据 |
| Realtime SFU、TURN、媒体适配器 | 有媒体需求才启用，验证独立音轨、音频格式和译音回传；不等同于 `ws` 库 |
| Issues、Logs、Traces、Alerts | Workers 服务端优先采用，按需开通、采样与上传 Source Map，不用于替代移动原生监控 |
| Analytics Engine、Zaraz | 分别承担业务指标和事件分发，不作精确财务账本 |
| Email Sending、Web Push | 按需接入事务邮件与浏览器通知，验证码校验和用户权限仍由应用负责；Web Push 的库与浏览器支持按接入验证确定 |




# 文件处理工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `unpdf` | 基于 PDF.js 的 PDF 处理工具，项目用于提取 PDF 简历文本。 |
| `mammoth` | DOCX 文档解析工具，支持提取纯文本或转换为 HTML，项目用于解析 Word 简历。 |


# 音视频录制工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `recordrtc` | 浏览器音视频录制库，封装媒体录制和录制结果处理能力。 |


# 表单与数据校验工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `react-hook-form` | React 表单管理库，处理字段状态、表单校验和提交交互。 |
| `zod` | Schema 定义与运行时校验库，用于校验 API 参数、环境变量和 AI 结构化输出，并推导 TypeScript 类型。 |

# 日期与时区工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `date-fns` | 提供不可变的日期计算、格式化和比较函数。 |
| `date-fns-tz` | 在 `date-fns` 基础上处理时区转换、时区格式化和跨时区业务时间。 |


# 样式与构建工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `tailwindcss` | 实用类 CSS 框架，通过组合样式类构建页面。 |
| `postcss` | CSS 处理工具，通过插件转换和生成样式。 |
| `@tailwindcss/postcss` | 将 Tailwind CSS 集成到 PostCSS 构建流程。 |
| `@vitejs/plugin-react` | Vite 的 React 插件，为基于 Vite 的开发、构建和测试工具提供 React 支持。 |



# 开发与代码检查工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `typescript` | 为 JavaScript 增加静态类型检查，帮助提前发现类型错误。 |
| `tsx` | 直接执行 TypeScript 文件，项目用于运行 Gateway 和 Worker。 |
| `eslint` | 静态代码检查工具，用于发现潜在错误和不规范写法。 |
| `eslint-config-next` | Next.js 推荐的 ESLint 规则集，包含框架及 React 相关检查。 |
| `@eslint/eslintrc` | ESLint 配置兼容工具，帮助传统配置接入新版配置体系。 |



# 自动化测试工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `vitest` | 单元测试和组件测试运行器，提供断言、Mock 和测试组织能力。 |
| `@testing-library/react` | 从用户视角测试 React 组件，例如点击按钮、填写表单和检查界面变化。 |
| `@testing-library/jest-dom` | 扩展 DOM 测试断言，例如判断元素是否可见、是否禁用。 |
| `jsdom` | 在 Node.js 中模拟浏览器 DOM 环境，支持组件测试。 |
| `@playwright/test` | 浏览器端到端测试框架，用于验证完整用户流程和页面表现。 |


# TypeScript 类型声明
| 第三方库 | 简要介绍 |
| --- | --- |
| `@types/node` | Node.js API 的类型声明。 |
| `@types/react` | React API、组件和 JSX 的类型声明。 |
| `@types/react-dom` | React DOM API 的类型声明。 |
| `@types/ws` | WebSocket 库 `ws` 的类型声明。 |
| `@types/recordrtc` | 录制库 `recordrtc` 的类型声明。 |
| `@types/qrcode` | 二维码库 `qrcode` 的 TypeScript 类型声明，提供接口参数、返回值及调用方式的类型检查。 |



