# Cloudflare 技术选型能力清单

**先看新项目要完成什么业务，再选择对应能力；按需组合，不必整套接入。** 本清单以 Workers Paid 为账号前提，整理两份参考文档涉及的能力，供新项目选型复用。
**每月 $5 是 Workers Paid 账号级起步费，不是所有产品的固定总费用。** [Workers 官方计费说明](https://developers.cloudflare.com/workers/platform/pricing/)。

| 项目 | 说明 |
| --- | --- |
| 适用范围 | 网站、Next.js 全栈应用、SaaS、AI 工具、内部系统等新项目 |
| 范围边界 | 覆盖参考文档中的主要选型能力，不是 Cloudflare 全产品目录，也不是账号已开通资源列表 |
| 使用方法 | 先查功能分组，再查业务组合；价格和额度统一查套餐文档，正式采用前复核官方限制与兼容性 |

## 1. 前端部署与页面交付

解决“网页、静态资源如何上线，用户通过什么地址访问”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Static Assets](https://developers.cloudflare.com/workers/static-assets/) | 托管并分发预先构建好的 HTML、JS、CSS、图片等静态文件 | 产品官网、前端 SPA、活动页、帮助站的公开资源 | 直接静态资源分发与需要运行 Worker 的请求应区分；私有文件另做授权 |
| [OpenNext](https://developers.cloudflare.com/workers/framework-guides/web-apps/opennext/) | 把 Next.js 构建结果适配为 Workers 可运行的部署产物 | 已有 Next.js 全栈项目要部署到 Cloudflare | 运行能力 / 工具；按项目版本核查兼容性，不是独立后端服务 |
| [vinext](https://developers.cloudflare.com/workers/framework-guides/web-apps/nextjs/) | 基于 Vite 实现 Next.js API 表面的部署路径 | 新项目评估 Cloudflare 官方推荐的 Next.js 应用路径 | 运行能力 / 工具；按参考资料仍属 beta，先检查兼容性，不与 OpenNext 同时作为同一项目部署适配方案 |
| [Pages](https://developers.cloudflare.com/pages/) | 网站部署产品，适合发布静态内容；动态 Functions 纳入 Workers 相关计费 | 独立文档站、静态官网、静态导出项目 | 静态托管规则与动态函数计费分开；全栈 Next.js 优先核查 Workers 的框架支持路径 |
| [Custom Domain / HTTPS](https://developers.cloudflare.com/workers/configuration/routing/custom-domains/) | 为 Worker 提供正式域名，并管理相关 DNS 记录和证书 | 官网、应用工作台、API 的正式访问地址 | 自定义域名能力；域名注册续费另算，HTTPS 不代替用户权限检查 |

OpenNext 和 vinext 放在本组，是为了按框架部署入口查找；它们也涉及服务端渲染和 API 部署，不是仅支持静态前端。实际执行后端逻辑的运行环境见下一组。

## 2. 后端运行与计算环境

解决“业务接口、服务端逻辑、后台程序在哪里执行”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Workers](https://developers.cloudflare.com/workers/) | 接收 HTTP 请求并执行业务代码，生成响应、访问数据库或调用外部 API | SaaS 工作台、订单接口、表单提交、支付回调、AI 请求入口 | 套餐额度；不自带登录、订单或支付系统，需要自行实现或集成 |
| [Containers](https://developers.cloudflare.com/containers/) | 运行容器镜像、后台程序及其系统依赖 | 音视频转换、依赖本地工具的文档处理、已有后台程序迁移 | 套餐含部分资源额度，运行超额另计；考虑启动时间、内存、磁盘和出站成本，临时磁盘不作永久业务存储 |
| [Node.js compatibility](https://developers.cloudflare.com/workers/runtime-apis/nodejs/) | 在 Workers 中提供部分 Node.js API 的兼容支持 | 复用项目已有服务端依赖、加密或数据处理模块 | 运行能力；不保证所有 npm 包、原生模块和系统二进制都能运行 |

容器提供独立运行环境，不能把它的运行费用与 Workers CPU 费用混为一项，见 [Containers 定价](https://developers.cloudflare.com/containers/platform/pricing/)。

## 3. 数据库、文件存储与数据库连接

解决“业务数据、文件内容分别放哪里，怎样访问数据库”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [D1](https://developers.cloudflare.com/d1/) | 托管 SQLite 语义的关系数据库，使用 SQL 表、索引和查询保存结构化数据 | 新工具的内容目录、用户偏好、轻量业务记录；其他领域需验证事务与容量要求 | 套餐额度；不自带 Supabase Auth 或同样的 PostgreSQL / RLS 体系，不是 PostgreSQL 的无改动替代 |
| [R2](https://developers.cloudflare.com/r2/) | 对象存储，按对象键保存文件内容，支持 S3 兼容接口、上传下载和私有访问设计 | 简历原文件、录音、图片、导入文件、备份 | 独立免费层与独立计费；业务数据库保存文件归属和元数据，应用签发短期访问凭证 |
| [Workers KV](https://developers.cloudflare.com/kv/) | 以键值方式保存小块数据，适合读取多、更新少且容许短暂陈旧的信息 | 公共配置缓存、展示文案、非敏感公共字典 | 套餐额度；最终一致，不用于扣余额、扣库存、支付记账或互斥锁 |
| [Hyperdrive](https://developers.cloudflare.com/hyperdrive/) | 为已有 PostgreSQL / MySQL 提供连接池与可配置查询缓存，减少反复建连开销 | Workers 使用原生数据库驱动访问远程数据库 | Workers Paid 包含相关能力；数据库本身仍另付费，不会给 Supabase HTTP SDK 自动加速；动态敏感查询需评估缓存 |
| [DO SQLite](https://developers.cloudflare.com/durable-objects/best-practices/access-durable-objects-storage/) | Durable Object 内部持久存储，保存该对象管理的状态 | 房间状态、资源协调进度、流程恢复信息 | 计入 Durable Objects 存储规则；不是一个让全站任意查询的独立 D1 数据库 |

关系数据和文件往往要组合：例如“文件库”由数据库保存所有者与文件记录，R2 保存文件字节。已有 PostgreSQL 能满足需求时，不因有 D1 额度就拆出第二套业务数据库。

## 4. 缓存与访问加速

解决“同样的公开内容，能否少计算、少查询、更快返回”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [CDN 缓存](https://developers.cloudflare.com/cache/) | 按缓存规则保存和分发可重复使用的响应，减少重复访问源服务 | 公开图片、帮助内容、公共资源反复访问 | 网站规则与相关服务计费分别核对；不是所有 HTML 或 API 都自动缓存 |
| [Cache API](https://developers.cloudflare.com/workers/runtime-apis/cache/) | 让 Worker 代码控制响应的缓存键、写入、读取和删除 | 公共接口缓存、按语言区分的公开内容缓存 | 运行能力；需设计有效期和失效策略，不当作永久存储或跨区域一致状态 |
| [ISR](https://developers.cloudflare.com/workers/framework-guides/web-apps/opennext/) | 框架按策略生成并更新页面，让内容不必每次访问都重新生成 | 文章、公开商品介绍、帮助中心 | Next.js 框架能力，依赖所选适配器及缓存配置；可能需要额外存储或重验证组件 |

| 内容类型 | 缓存判断 |
| --- | --- |
| 公共内容，允许短暂旧版本 | 可以评估 CDN、Cache API、ISR 或 KV，按缓存对象选择 |
| 当前余额、支付状态、一次性授权 | 读取可信当前状态，不能用公共缓存承担业务判断 |
| 用户简历、私有报告、订单详情 | 不能进入跨用户共享缓存；确需私有缓存必须定义身份隔离与失效规则 |

## 5. 异步任务、定时任务与流程编排

解决“用户提交后先返回，后台慢慢做；定时检查；多步失败后继续做”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Queues](https://developers.cloudflare.com/queues/) | 生产者投递消息，消费者异步处理，支持批量、重试、延迟和死信 | 下单后通知、批量导入、任务削峰、外部事件处理 | 套餐额度；消息可能重复，消费逻辑仍须幂等；已有可靠任务队列时优先复用 |
| [Workflows](https://developers.cloudflare.com/workflows/) | 持久化多步骤流程，支持等待外部事件、睡眠、重试和状态恢复 | 外部转码任务提交→等待结果→保存→通知；需要人工确认的内容流程 | 套餐额度并按相关维度计费；解决步骤推进，不代替容器运行系统工具 |
| [Cron Triggers](https://developers.cloudflare.com/workers/configuration/cron-triggers/) | 按 Cron 表达式触发 Worker 的定时处理入口 | 定时巡检、检查到期任务、清理临时数据、定期同步 | 运行能力，执行按相关 Workers 规则计量；使用 UTC，不是精确实时调度器 |
| [DO Alarms](https://developers.cloudflare.com/durable-objects/api/alarms/) | 为某个 Durable Object 设置未来触发时间 | 对某房间、资源或协调流程做后续检查和清理 | 计入 DO 相关请求与存储；需实现幂等，不代替通用任务队列 |

| 面对的需求 | 优先判断 |
| --- | --- |
| “这件事稍后做” | Queues 或现有持久化任务队列 |
| “先做 A，等待 B，再做 C” | Workflows 或现有阶段编排 |
| “每隔一段时间看看有没有事做” | Cron Triggers |
| “这个对象过一会儿再检查” | DO Alarms |
| “需要运行系统工具或自定义后台程序” | Containers 提供执行环境；调度与执行是两个职责 |

## 6. 实时协作、状态协调与内部通信

解决“多人共享同一份实时状态”和“自己的服务之间如何调用”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Durable Objects](https://developers.cloudflare.com/durable-objects/) | 为指定对象提供稳定身份、状态与协调执行入口，可结合 WebSocket | 多人房间、协作会话、游戏房间状态、资源生命周期协调 | 套餐额度；按对象设计分片与持久化，不能自动保证跨多个对象的业务事务 |
| [Service Bindings / RPC](https://developers.cloudflare.com/workers/runtime-apis/bindings/service-bindings/) | 一个 Worker 通过绑定调用另一个 Worker 的函数或请求入口，无需公开 URL | Web 通知后台调度、共用内部计算服务、拆分已有独立服务 | 运行能力；内部调用不额外增加 Worker 请求费，相关 CPU、DO 等仍计量 |
| OpenNext 自引用绑定 | 适配器内部调用自身 Worker 所需的基础配置 | 按 OpenNext 部署要求提供 `WORKER_SELF_REFERENCE` | 框架实现细节，不是新业务优先选择的独立产品；不能据此判断已启用 ISR |

实时聊天、协作与实时音视频是不同需求。DO 可参与消息和状态协调，但不因此自动具备完整音视频平台、媒体转发或模型实时语音能力。音视频项目应按媒体协议、处理方式和服务商能力单独选择实现。

## 7. AI 推理、模型调用管理与语义搜索

解决“让模型生成内容、管理模型调用、从资料中按意思查找”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Workers AI](https://developers.cloudflare.com/workers-ai/) | 调用 Cloudflare 提供的模型执行推理，实际输入输出能力由模型决定 | 文本摘要、分类、内容生成、生成向量等 | 独立每日免费量及超额推理计费；先验证质量、语言、延迟和模型授权，不把它等同于任何外部供应商模型 |
| [AI Gateway](https://developers.cloudflare.com/ai-gateway/) | 在模型调用前增加观察和管理入口，提供统计、缓存及路由等能力 | 多模型调用管理、供应商失败排查、统一观察成本和延迟 | 核心能力有免费规则，日志及部分附加能力另核价；它不是模型，先核查目标供应商和协议支持 |
| [Vectorize](https://developers.cloudflare.com/vectorize/) | 保存模型生成的向量，检索语义相近的内容 | 知识库检索、语义搜索、资料相似推荐、RAG | 套餐相关额度与超额计费单独核对；需要向量生成、原文存储和权限过滤，不会单独完成知识问答 |

| 要做的 AI 功能 | 常见组合 | 分工 |
| --- | --- | --- |
| 一次文本生成 | Workers + Workers AI 或外部模型 API | Workers 校验并编排，模型生成 |
| 文件摘要 | R2 + 解析执行环境 + 模型；需要异步时加队列或流程 | R2 存原文件，代码提取文本，模型生成摘要 |
| 知识库问答 | R2 / 数据库 + 向量模型 + Vectorize + 生成模型 | 保存原文、生成向量、检索相关片段、结合资料生成回答 |
| 管理多模型调用 | 已有模型客户端 + AI Gateway | 先验证兼容性再引入管理入口；不默认代理全部 WebRTC 音频链路 |

以上组合是通用选型示例，模型供应商按新项目需求选择。AI 输入、输出和调用日志应按用户授权、数据隔离及脱敏要求设计。

## 8. 安全、人机验证与身份门禁

解决“拦攻击、减轻流量冲击、识别自动化访问、限制内部应用访问”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [WAF](https://developers.cloudflare.com/waf/) | 根据规则检查 Web / API 请求并过滤攻击特征，支持相应规则配置 | 恶意扫描、常见攻击请求、对指定敏感入口设置规则 | 网站套餐规则；权限与额度单独核对，不能代替资源归属校验 |
| [DDoS 防护](https://developers.cloudflare.com/ddos-protection/) | 缓解大量恶意流量造成的服务冲击 | 公共网站、API 面对流量攻击 | 网站和产品防护规则；不等于免费处理无限正常业务，也不覆盖所有付费接口滥用 |
| [Turnstile](https://developers.cloudflare.com/turnstile/) | 人机验证，增加自动化滥用成本，由服务端验证结果 | 注册刷号、表单垃圾提交、登录入口滥用 | 独立免费层；不能替代登录、业务授权或全部风控 |
| [Cloudflare Access](https://developers.cloudflare.com/cloudflare-one/access-controls/) | 进入内部应用前执行基于身份与策略的访问检查 | 员工运营后台、内部工具、测试管理入口 | Zero Trust 体系，按身份套餐核对；不等同于面向客户的产品账号系统 |
| [Zero Trust](https://developers.cloudflare.com/cloudflare-one/) | 面向内部资源访问的安全产品体系，Access 是其中一部分 | 企业员工与内部系统的访问管理 | 网站 / 身份套餐；不是每月起步费附赠的完整企业安全套件 |
| [Workers Secrets / Variables](https://developers.cloudflare.com/workers/configuration/secrets/) | 为部署提供密钥和普通环境配置；运行时代码读取所需值 | 支付密钥、模型 API Key、测试与生产配置隔离、功能开关 | 运行能力；不是用户密码数据库，不把私密值放进浏览器公开变量 |

| 需要判断的事 | 对应机制 |
| --- | --- |
| 请求是否匹配攻击规则 | WAF |
| 提交者是否像真人 | Turnstile |
| 是否允许员工进入内部应用 | Access |
| 用户是谁、是否购买会员、是否拥有资源 | 产品登录与业务授权系统，例如 Supabase Auth + 应用权限 / RLS |
| 支付通知是否可信 | 支付平台验签、事件去重、金额校验与事务入账，不由 WAF 代办 |

## 9. 图片与视频业务

解决“文件存起来之后，怎样加工并适合播放或展示”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Images](https://developers.cloudflare.com/images/) | 图片缩放、格式转换、优化与相应托管分发 | 商品缩略图、头像、按设备尺寸显示图片 | 独立免费层和付费计划；R2 存图不自动获得全部图片处理能力 |
| [Stream](https://developers.cloudflare.com/stream/) | 托管视频上传、存储、编码和直播 / 点播分发 | 在线课程、会员视频、用户视频上传、直播功能 | 独立计费；视频内容访问仍需授权设计，不是纯 MP3 文件回放的必需产品 |

| 业务需求 | 优先选择 |
| --- | --- |
| 只保存并下载文件，或播放已有 MP3 | R2 + 应用授权 + 浏览器支持的播放方式 |
| 同一图片需要多个尺寸和优化格式 | Images；原图存储按方案选择 |
| 视频要转码、多网络条件播放、托管播放器链路 | Stream |
| 自定义音视频处理步骤 | Containers 承载自定义处理程序，R2 保存文件；需要时结合 Queues / Workflows 调度 |

## 10. 监控、统计、开发与成本管理

解决“能不能发布、哪里出错、用户访问怎样、费用是否失控”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Logs / Observability](https://developers.cloudflare.com/workers/observability/logs/workers-logs/) | 收集、查询运行日志、错误与诊断线索 | 支付回调失败、请求报错、任务失败、容器运行排查 | 套餐相关额度，规则可能变化；不记录密钥、JWT、完整私密内容，不以日志代替账本 |
| [Web Analytics](https://developers.cloudflare.com/web-analytics/) | 观察网站访问、来源和页面性能 | 官网流量、内容访问、页面体验评估 | 独立可用能力；不自动生成完整注册付费漏斗，业务事件需要另设计 |
| [Wrangler](https://developers.cloudflare.com/workers/wrangler/) | 开发、部署、管理 Worker、绑定、密钥并查看日志 | 本地调试、版本发布、环境管理 | 开发工具；执行部署会变更资源，生产发布需对应验证 |
| [workerd](https://github.com/cloudflare/workerd) | Workers 使用的运行时，可用于本地执行与兼容性验证 | 检查仅在 Workers 中出现的 API 或依赖问题 | 运行工具；本地替身测试不能证明生产外部服务和网络已通过验收 |
| [费用提醒 / Spending limits](https://developers.cloudflare.com/workers/platform/spend-management/) | 提醒费用增长，并可对支持产品设置达到阈值后的调用限制 | 新项目上线、测试流量异常、成本保护 | Workers 与 DO 等分开核对；不是全产品精确总费用封顶，阻断会影响业务可用性 |

## 11. 按新项目业务快速组合

以下是起步组合，数据库和身份系统可复用已有方案。只有业务需要时才增加后面的能力。

| 新项目类型 | 起步能力组合 | 随需求增加 | 需要额外解决的业务部分 |
| --- | --- | --- | --- |
| 静态官网 / 活动页 | Static Assets 或 Pages + 域名 / HTTPS | Web Analytics、Turnstile 表单验证 | 联系表单保存、通知发送和用户授权 |
| 登录后的 SaaS | Workers + 所选框架路径 + 数据库 + 身份系统 | R2、Turnstile、Logs、公共内容缓存 | 账号、租户隔离、角色权限、业务数据规则 |
| 小型内容工具 | Workers + D1 | R2、公共内容缓存 | 用户授权与数据访问规则 |
| 文件管理 / 文件转换 | Workers + R2 + 数据库 | Queues / Workflows；需要系统工具时加 Containers | 文件所有权、格式校验、上传限制、删除与任务状态 |
| AI 摘要 / 写作工具 | Workers + Workers AI 或外部模型 | R2、异步任务、AI Gateway | 额度、输入输出校验、模型质量和成本控制 |
| 知识库问答 | Workers + 原文存储 + 向量生成 + Vectorize + 生成模型 | 数据导入队列、流程编排、模型调用管理 | 文档权限、检索隔离、答案质量与引用设计 |
| 多人协作 / 房间应用 | Workers + Durable Objects + 持久状态 | WebSocket、DO Alarms、业务数据库 | 身份、房间权限、重连、消息顺序和冲突处理 |
| 视频课程 / 会员内容 | Workers + Stream + 数据库 + 身份系统 | Images、Web Analytics | 购买和观看权限、支付验签、课程管理 |
| 内部管理工具 | Workers + 数据库 + Access | Logs、Cron | 后台业务角色、审计和资源授权 |
| 支付型应用 | Workers + 业务数据库 + 支付渠道 | 异步通知处理、Logs、费用提醒 | 订单状态、签名验证、幂等入账；Cloudflare 不提供完整收款业务 |

## 12. 实施前的选型检查

| 要回答的问题 | 判断方式 |
| --- | --- |
| 是否已有可复用方案 | 已有数据库、队列、身份系统能满足需求时，先扩展已有能力 |
| 数据属于哪一种 | 结构化业务数据、文件、缓存、对象协调状态分别选择，不混用 |
| 是同步请求还是后台工作 | 用户需要立即结果时走请求；可延后、耗时或需重试时设计异步 |
| 是否依赖系统程序 | 依赖系统工具、原生二进制或完整进程环境时评估 Containers |
| 是否允许旧数据 | 能接受短暂陈旧才评估共享缓存；支付、余额等依赖可信状态 |
| 是否属于敏感资产 | 先设计授权和隔离，再考虑缓存、签名、日志或模型调用 |
| 套餐是否覆盖 | 检查产品资格、额度、资源上限、独立计费以及额外第三方账单 |
| 是否适合目标用户地区 | 对真实用户到应用、数据库、对象存储和模型的链路测延迟，不只看节点分布 |
| 是否经过运行时验证 | 检查框架版本、兼容标记、依赖与绑定，本地测试后做隔离平台验收 |


