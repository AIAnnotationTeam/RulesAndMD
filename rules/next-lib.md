

Next全栈框架的常用第三方库


# 应用框架与界面

| 第三方库 | 简要介绍 |
| --- | --- |
| `next` | 基于 React 的全栈框架，提供页面路由、服务端渲染、HTTP API 和项目构建能力。 |
| `react` | 组件化界面库，负责页面展示、用户交互和状态管理。 |
| `react-dom` | 将 React 组件渲染到浏览器页面，并提供服务端渲染能力。 |


# 网络通信工具

| 第三方库 | 简要介绍 |
| --- | --- |
| `axios` | 基于 Promise 的 HTTP 客户端，支持浏览器和 Node.js，用于调用 HTTP API，提供请求与响应拦截器、超时配置和请求取消等能力。 |
| `ws` | Node.js WebSocket 库，用于实时网关与浏览器、AI 供应商之间的双向通信。 |



# 身份认证与数据库工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `@supabase/supabase-js` | Supabase JavaScript SDK，提供用户认证、数据库查询和数据库函数调用等能力。 |
| `@supabase/ssr` | 为服务端渲染应用提供 Supabase 会话管理支持，通过 Cookie 衔接浏览器和服务端登录状态。 |


# 对象存储工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `@aws-sdk/client-s3` | AWS S3 客户端，项目用它访问兼容 S3 协议的 Cloudflare R2，存储和管理录音文件。 |
| `@aws-sdk/s3-request-presigner` | 生成带有效期的上传、下载签名 URL，让浏览器安全访问私有存储文件。 |


# 支付工具
| 第三方库 | 简要介绍 |
| --- | --- |
| `stripe` | Stripe 服务端 SDK，用于创建支付流程、查询支付数据和验证 Webhook。 |
| `alipay-sdk` | 支付宝服务端 SDK，用于构造支付请求、签名和验证支付通知。 |




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




