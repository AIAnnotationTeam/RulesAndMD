# Cloudflare 技术选型能力清单

**先看新项目要完成什么业务，再选择对应能力；按需组合，不必整套接入。** 本清单以 Workers Paid 为账号前提，整理 Cloudflare 官方能力及必要的外部集成，供新项目选型复用。
**每月 $5 是 Workers Paid 账号级起步费，不是所有产品的固定总费用。** [Workers 官方计费说明](https://developers.cloudflare.com/workers/platform/pricing/)。

| 项目 | 说明 |
| --- | --- |
| 适用范围 | 网站、Next.js 全栈应用、SaaS、AI 工具、内部系统等新项目 |
| 范围边界 | 覆盖常用业务及补充核查的选型能力，不是 Cloudflare 全产品目录，也不是账号已开通资源列表；Better Auth、支付宝、Creem 等明确标记为外部集成 |
| 使用方法 | 先查功能分组，再查业务组合；价格和额度统一查套餐文档，正式采用前复核官方限制与兼容性 |
| 补充核查日期 | 2026-11-25，北京时间；新增监控、统计、通知、邮件和支付相关能力，未执行真实部署、投递或交易测试 |

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

## 6. 实时音视频、协作状态与内部通信

解决“多人音视频如何转发、受限网络如何连接、媒体怎样接入 AI”，以及“共享状态和内部服务如何协调”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Realtime SFU](https://developers.cloudflare.com/realtime/sfu/) | 在端点之间转发 WebRTC 音频、视频和 DataChannel，应用决定发布者与订阅者 | 多人会议、互动音视频、需要直接控制音轨路由的应用 | Realtime 独立计费，不纳入 Workers 起步费；应用自行实现信令、身份、房间权限和订阅控制，不自带模型翻译 |
| [Realtime TURN](https://developers.cloudflare.com/realtime/turn/) | 当 NAT 或防火墙阻碍直接通信时，为 WebRTC 提供中继连接路径 | 受限网络中的会议连通性、已有 WebRTC 应用的 TURN 补充 | TURN 是连通性中继，不能代替 SFU 的多人音轨路由；与 Realtime SFU 配合的流量避免重复计费，单独使用按 TURN 规则计量 |
| [WebSocket 媒体适配器](https://developers.cloudflare.com/realtime/sfu/features/media-transport-adapters/websocket-adapter/) | 将 SFU 单条音轨接到应用的 WebSocket 服务，导出音频或将生成音频发布回 SFU；也支持视频以 JPEG 帧导出 | 实时转写、AI 语音处理、百炼同传链路的适配验证 | 每个适配器为单轨单向，双向音频需分别创建；应用须提供公开 WSS 服务并适配协议，不能直接认定兼容百炼接口；流量纳入 Realtime 计费 |
| [Durable Objects](https://developers.cloudflare.com/durable-objects/) | 为指定对象提供稳定身份、状态与协调执行入口，可结合 WebSocket | 多人房间、协作会话、游戏房间状态、资源生命周期协调 | 套餐额度；按对象设计分片与持久化，不能自动保证跨多个对象的业务事务 |
| [Service Bindings / RPC](https://developers.cloudflare.com/workers/runtime-apis/bindings/service-bindings/) | 一个 Worker 通过绑定调用另一个 Worker 的函数或请求入口，无需公开 URL | Web 通知后台调度、共用内部计算服务、拆分已有独立服务 | 运行能力；内部调用不额外增加 Worker 请求费，相关 CPU、DO 等仍计量 |
| OpenNext 自引用绑定 | 适配器内部调用自身 Worker 所需的基础配置 | 按 OpenNext 部署要求提供 `WORKER_SELF_REFERENCE` | 框架实现细节，不是新业务优先选择的独立产品；不能据此判断已启用 ISR |

实时聊天、协作状态与音视频媒体传输应分别选型：DO 管状态，SFU 转发媒体，TURN 补充连通性，WebSocket 适配器连接媒体与处理服务。标准会议 SDK 与界面组件还可评估官方 [RealtimeKit](https://developers.cloudflare.com/realtime/realtimekit/)；需要直接控制音轨与信令时再评估 SFU。

以下为接入外部语音模型的工程验证路径，不代表已经完成百炼兼容性或性能测试。

| 接入环节 | 需要确认的边界 |
| --- | --- |
| 获取发言音轨 | 按发言者订阅独立音轨，避免将多人混音直接作为同一个翻译来源 |
| SFU → 处理网关 | 适配器由 SFU 主动连接应用 WSS；音频使用 Protobuf 消息封装的 48 kHz、双声道、16-bit 小端 PCM，不是百炼原生事件格式 |
| 处理网关 → 百炼 | 由自建网关解析封包、按所选模型转换采样率和声道、维护鉴权与模型事件；模型选择见[百炼能力清单](百炼平台能力清单.md) |
| 译音返回 | 按适配器格式转换后通过另一适配器发布回 SFU，或由客户端独立播放；验证字幕归属、译音顺序、重连和全链路延迟 |
| 视频适配 | 支持 JPEG 帧导出，不支持视频导入；不要将音频回灌能力扩大为任意视频双向适配 |
| 费用与地域 | SFU、TURN 与适配器按 [Realtime 定价](https://developers.cloudflare.com/realtime/sfu/platform/pricing/)核算共享免费量和出站流量，网关计算与百炼调用另计；TURN 官方注明不含 Cloudflare 中国网络，需实测目标地区链路 |

| 本组补充核查 | 说明 |
| --- | --- |
| 核查日期 | 2026-11-19，北京时间；依据上述官方能力和定价文档，未执行真实音视频或百炼联调 |

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
| 语音合成与音色选择（外部集成） | Workers + 百炼 TTS；详见 [百炼语音合成能力](百炼平台能力清单.md#9-语音合成配音声音复刻与声音设计) | 百炼生成音频并提供音色，Workers 承担鉴权、协议适配与交付；模型、音色和流式方式按百炼清单核对，不计作 Cloudflare 自有模型 |
| 文件摘要 | R2 + 解析执行环境 + 模型；需要异步时加队列或流程 | R2 存原文件，代码提取文本，模型生成摘要 |
| 知识库问答 | R2 / 数据库 + 向量模型 + Vectorize + 生成模型 | 保存原文、生成向量、检索相关片段、结合资料生成回答 |
| 管理多模型调用 | 已有模型客户端 + AI Gateway | 先验证兼容性再引入管理入口；不默认代理全部 WebRTC 音频链路 |

以上组合是通用选型示例，模型供应商按新项目需求选择。AI 输入、输出和调用日志应按用户授权、数据隔离及脱敏要求设计。

## 8. 安全、人机验证与用户登录

解决“拦攻击、识别自动化访问、用户如何登录、谁能进入应用”的问题。登录证明身份，业务授权决定该身份能操作哪些资源，两者分别设计。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [WAF](https://developers.cloudflare.com/waf/) | 根据规则检查 Web / API 请求并过滤攻击特征，支持相应规则配置 | 恶意扫描、常见攻击请求、对指定敏感入口设置规则 | 网站套餐规则；权限与额度单独核对，不能代替资源归属校验 |
| [DDoS 防护](https://developers.cloudflare.com/ddos-protection/) | 缓解大量恶意流量造成的服务冲击 | 公共网站、API 面对流量攻击 | 网站和产品防护规则；不等于免费处理无限正常业务，也不覆盖所有付费接口滥用 |
| [Turnstile](https://developers.cloudflare.com/turnstile/) | 人机验证，增加自动化滥用成本，由服务端验证结果 | 注册刷号、表单垃圾提交、登录入口滥用 | 独立免费层；不能替代登录、业务授权或全部风控 |
| [Cloudflare Access](https://developers.cloudflare.com/workers/configuration/cloudflare-access/) | 提供登录入口、身份验证及访问策略，在请求到达应用前检查身份；受 Access 认证的 Worker 可读取用户身份 | 员工后台、合作伙伴门户、邀请制或其他受控访问应用 | Zero Trust 体系，按身份套餐和用户规模核对；不限于内部员工，但不自动提供产品用户资料、会员权益与资源归属管理 |
| [Access 邮箱验证码登录](https://developers.cloudflare.com/cloudflare-one/integrations/identity-providers/one-time-pin/) | 向策略允许的邮箱发送一次性验证码，无需额外集成第三方身份提供商 | 邀请制工具、按邮箱或域名限制访问的门户 | Access 登录方式，不是任意业务邮件发送服务；允许哪些邮箱由策略决定，应用仍需维护业务用户与权限 |
| [Access Passkeys](https://developers.cloudflare.com/cloudflare-one/integrations/identity-providers/passkeys/) | 用户登记通行密钥后，通过设备、安全密钥或密码管理器登录 | 希望使用无密码登录的受保护应用 | 依赖 WebAuthn 与设备支持；密钥登记、登录成功不自动赋予访问权，仍须匹配 Access 策略 |
| [Access 身份提供商集成](https://developers.cloudflare.com/cloudflare-one/integrations/identity-providers/) | 接入组织或外部身份提供商，可同时配置多个登录来源 | 企业单点登录、合作伙伴身份接入 | 身份源、组信息和登录方式按对应集成检查；不同登录方式不保证携带相同身份属性 |
| [Cloudflare 账号 OAuth](https://developers.cloudflare.com/fundamentals/oauth/integrate-with-cloudflare/) | 注册 OAuth 客户端，让用户通过已有 Cloudflare 账号授权应用 | Cloudflare 开发者工具、需要用户授权其 Cloudflare 资源的应用 | 面向已有 Cloudflare 账号；客户端可见范围和授权 Scope 按官方配置，不等同于为任意产品托管一套独立账号库 |
| [Workers 上部署 Better Auth](https://better-auth.com/docs/guides/cloudflare) + [D1 支持](https://better-auth.com/blog/1-5) | 在 Workers 运行认证框架，用 D1 保存用户与会话等认证数据，构建产品自己的账号系统 | 面向公众的 SaaS、自定义注册登录流程、希望将认证运行在 Cloudflare 的项目 | 第三方开源框架部署方案，不是 Cloudflare 原生托管 Auth 产品；运行与存储按 Workers / D1 计费，应用负责框架维护、迁移、邮件发送及业务授权，按目标版本验证兼容性 |
| [Zero Trust](https://developers.cloudflare.com/cloudflare-one/) | 面向内部资源访问的安全产品体系，Access 是其中一部分 | 企业员工与内部系统的访问管理 | 网站 / 身份套餐；不是每月起步费附赠的完整企业安全套件 |
| [Workers Secrets / Variables](https://developers.cloudflare.com/workers/configuration/secrets/) | 为部署提供密钥和普通环境配置；运行时代码读取所需值 | 支付密钥、模型 API Key、测试与生产配置隔离、功能开关 | 运行能力；不是用户密码数据库，不把私密值放进浏览器公开变量 |

| 需要判断的事 | 对应机制 |
| --- | --- |
| 请求是否匹配攻击规则 | WAF |
| 提交者是否像真人 | Turnstile |
| 是否允许员工、合作伙伴或受邀用户进入应用 | Access + 对应登录方式 + 访问策略 |
| 是否仅需用验证码验证邮箱并控制入口 | Access 邮箱验证码登录；业务账号和资源授权另行判断 |
| 是否用已有 Cloudflare 账号登录并授权 | Cloudflare 账号 OAuth；确认目标用户具备 Cloudflare 账号 |
| 是否需要公众产品自己的注册登录与会话 | 优先评估 Workers + D1 + Better Auth；已有身份服务满足需求时先复用 |
| 是否购买会员、是否拥有某条资源 | 应用内权益、角色与资源归属校验；Access 或认证框架登录成功不代表拥有业务权限 |
| 支付通知是否可信 | 支付平台验签、事件去重、金额校验与事务入账，不由 WAF 代办 |

原生 Access 登录、Cloudflare 账号 OAuth、第三方认证框架部署是三种不同路径。新项目按业务选择；不能把 Turnstile 当成登录，也不能把 D1 当成完整 Auth 服务。Workers + D1 + Better Auth 是能力组合建议，替换现有 Supabase Auth / Firebase Auth 前需核对登录方式、账号迁移、会话、移动端接入与权限模型。

| 本组登录能力补充核查 | 说明 |
| --- | --- |
| 核查日期 | 2026-11-25，北京时间；依据各行官方文档，未进行真实登录部署或账号迁移测试 |

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
| [Workers Issues](https://developers.cloudflare.com/workers/observability/issues/) | 检测 Workers 生产错误、聚合同类问题，结合 Source Map 定位源代码并触发自动通知 | Workers 服务端异常跟踪、重复错误排查 | 可优先评估替代服务端部分 Sentry 职责；需启用并上传构建生成的 Source Map，不覆盖移动端原生崩溃、ANR 和客户端完整诊断 |
| [Workers Traces](https://developers.cloudflare.com/workers/observability/traces/) / [Cloudflare Traces](https://developers.cloudflare.com/observability/traces/) | 查看 Worker 与外部调用，以及请求穿过规则、缓存、路由和源站的路径与耗时 | 慢请求、模型调用延迟、缓存与源站性能诊断 | Worker 与域名追踪按对应入口启用；按采样、保留期和 Observability 计费核算，不等于移动端启动、卡顿与完整客户端 APM |
| [OpenTelemetry 导出](https://developers.cloudflare.com/workers/observability/opentelemetry-export/) | 将 Workers 日志与链路发送到外部 OTLP 接收端 | 已有 Sentry、Grafana 等监控平台，需要关联服务端诊断 | 是导出集成，不代表 Cloudflare 接管接收端全部功能；日志和 Traces 的保存、导出及外部供应商费用分别核对 |
| [Alerts / Monitors](https://developers.cloudflare.com/notifications/) | 按平台事件或定时查询条件触发告警，通过邮件、Webhook 等渠道通知 | 错误率、服务异常、资源与费用告警 | 查询监测、检测方式和通知渠道按权限与套餐核对；这是运维告警，不是手机系统推送服务 |
| [Web Analytics](https://developers.cloudflare.com/web-analytics/) | 观察网站访问、来源和页面性能 | 官网流量、内容访问、页面体验评估 | 独立可用能力；不自动生成完整注册付费漏斗，业务事件需要另设计 |
| [Workers Analytics Engine](https://developers.cloudflare.com/analytics/analytics-engine/) | 从 Workers 写入自定义事件，通过 SQL 查询高基数业务指标 | 注册、功能使用、按用户统计调用量、服务健康和用量分析 | 应用设计采集与报表；检查采样及查询语义，不用统计聚合代替精确支付账本或交易裁决 |
| [Zaraz](https://developers.cloudflare.com/zaraz/) | 管理网站的统计工具、广告像素和第三方脚本，支持事件触发与同意管理 | 网站营销工具整合、转化事件分发 | 相关第三方工具与费用独立核对，不等于自带全部广告分析平台或完整移动安装归因 |
| [Zaraz HTTP Events API](https://developers.cloudflare.com/zaraz/http-events-api/) | 从服务端、移动端等无法运行 Web API 的环境发送业务事件 | 注册成功、交易完成、移动端行为的事件转发 | 需配置端点并设计防滥用、授权与数据最小化；接收事件不等于替代 Meta SDK 的登录、安装归因等全部功能 |
| [Wrangler](https://developers.cloudflare.com/workers/wrangler/) | 开发、部署、管理 Worker、绑定、密钥并查看日志 | 本地调试、版本发布、环境管理 | 开发工具；执行部署会变更资源，生产发布需对应验证 |
| [workerd](https://github.com/cloudflare/workerd) | Workers 使用的运行时，可用于本地执行与兼容性验证 | 检查仅在 Workers 中出现的 API 或依赖问题 | 运行工具；本地替身测试不能证明生产外部服务和网络已通过验收 |
| [费用提醒 / Spending limits](https://developers.cloudflare.com/workers/platform/spend-management/) | 提醒费用增长，并可对支持产品设置达到阈值后的调用限制 | 新项目上线、测试流量异常、成本保护 | Workers 与 DO 等分开核对；不是全产品精确总费用封顶，阻断会影响业务可用性 |

| 业务边界 | 选型判断 |
| --- | --- |
| Workers 错误、请求耗时与服务端告警 | 优先评估 Issues + Traces + Alerts，按需增加日志与 OTel 导出 |
| iOS / Android 原生崩溃、ANR、符号解析 | 当前清单未核实到完整替代能力，保留 Sentry / Crashlytics 等必要客户端方案 |
| 网站访问统计、自定义指标、广告事件分发 | 分别选择 Web Analytics、Analytics Engine、Zaraz，不把三者视为同一产品 |

## 11. 用户通知、邮件发送与邮件接收

解决“业务发生后怎样通知用户，以及应用如何发送和接收邮件”的问题。

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Web Push 官方实现方案](https://developers.cloudflare.com/agents/agent-class/communication-channels/webhooks/push-notifications/) | 由 Agent / Durable Object 保存订阅、调度，通过第三方 `web-push` 库和浏览器 Service Worker 投递通知 | 浏览器 / PWA 提醒、用户关闭标签页后的业务通知 | 平台组合与代码实现方案，不是独立托管推送产品；需权限、VAPID 和浏览器推送服务，受设备、网络与浏览器限制，不替代原生 APNs、FCM 或厂商通道 |
| [Email Service / Email Sending](https://developers.cloudflare.com/email-service/) | 通过 Workers binding、REST API 或 SMTP 发送事务邮件 | 注册验证、登录验证码、密码重置、订单确认与业务提醒 | 当前 Sending 标注 Beta；须配置发送域名，任意收件人发送需要 Workers Paid，Free 限账号内已验证收件地址；额度和费用另核对，不用于营销群发 |
| [Email Routing](https://developers.cloudflare.com/email-service/get-started/route-emails/) | 将域名收到的邮件转发到目标邮箱或交由 Worker 处理 | 客服入口、入站邮件处理、邮件驱动业务 | 是接收与路由能力，不等于通用邮箱客户端或完整邮件托管；与主动发送分开配置 |

| 业务需求 | 优先组合与边界 |
| --- | --- |
| 自部署 Auth 需要发验证邮件 | Better Auth + Email Sending；验证码生成、校验、有效期、次数限制和用户状态由认证逻辑负责 |
| 网站用户需要异步提醒 | Workers / Agent + 订阅存储 + Web Push；按浏览器环境验证送达 |
| 原生 App 需要系统级远程推送 | Workers 承接发送接口与调度，继续接 APNs、FCM 或厂商推送；不使用 WebSocket 冒充离线推送 |
| 运维人员需要异常告警 | 使用上一组 Alerts；不与产品用户通知混为一套通道 |

发送到账号内已验证地址与发送到任意用户地址的资格不同。正式发送前按[发送指南](https://developers.cloudflare.com/email-service/get-started/send-emails/)验证域名与收件范围，并按[官方 FAQ](https://developers.cloudflare.com/email-service/reference/faq/)处理事务邮件用途和投递限制。

## 12. 支付、订阅与专用付费访问

解决“谁向谁付款、通过什么渠道收款、付款后如何交付权益”的问题。**国内普通业务支付优先使用支付宝**；支付宝与 Creem 为外部支付集成，不是 Cloudflare 自有支付产品。

### 12.1 普通业务支付：外部渠道 + Cloudflare 后端

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [支付宝](https://b.alipay.com/home.htm)（国内支付优先） | 通过签约产品接入电脑网站、手机网站、App 等支付能力 | 国内网站订单、SaaS 套餐购买、服务费用支付 | 外部支付渠道，商户开通、产品资格与费率按支付宝核对；按实际终端选择产品，不能把所有扫码交互都当成当面付；Cloudflare 只承接订单、回调与权益业务 |
| [Creem](https://docs.creem.io/getting-started/quickstart)（海外普通支付方案） | 通过托管收银台接入一次性购买与订阅，作为 Merchant of Record 处理交易相关税务与结算 | 获准经营的软件、SaaS 和数字产品出海 | 外部 MoR 平台；先核对商户审核、产品适用性、地区和结算条件，不因为 Worker 部署全球就认定已获准收款 |
| Workers + 业务数据库 + 支付 Adapter | 服务端创建订单、调用支付平台、处理通知、查询状态与交付权益 | 支付宝和 Creem 的统一业务接入 | 不改变实际支付渠道；密钥只在服务端，Node.js SDK 在 Workers 的兼容性需实测，必要时按官方 API 或容器适配 |

| 支付场景 | 固定选型原则 |
| --- | --- |
| 国内 Web 业务收款 | 优先支付宝，电脑网站与手机网站分别选择已签约的适用产品 |
| 海外 Web 业务收款 | 采用 Creem，完成商户审核并确认地区、商品与结算支持 |
| 定期收费与会员权益 | 区分一次付费开通会员、到期手动续费和自动扣费；不能把一次支付当成自动订阅，自动扣费须具备对应产品与用户授权 |
| App 内数字内容购买 | 按目标商店、地区与业务类型核对内购规则；不以“国内优先支付宝”自动替代 StoreKit、Google Play Billing 或订阅管理服务 |
| 支付状态确认 | 服务端验签、核对订单与金额、幂等更新，并用可信查询补偿；浏览器跳回成功页不能作为入账依据 |

支付宝服务端接入见[官方 Node.js SDK](https://github.com/alipay/alipay-sdk-nodejs-all)，Creem 事件处理见[官方 Webhook 文档](https://docs.creem.io/code/webhooks)。订单、退款与权益状态保存在可信业务数据库，Queues 可用于可靠处理后续通知；未经验证的支付事件不能直接发放权益。

| Creem 接入边界 | 项目实施要求 |
| --- | --- |
| 付款渠道 | [官方付款方式](https://docs.creem.io/merchant-of-record/finance/payment-methods) 按产品和客户条件展示；支付宝仅支持一次性付款，不用于自动续费；国内支付宝直连继续独立接入 |
| 交易与结算 | MoR 交易处理与商家结算分开核查，费用和周期以 [结算文档](https://docs.creem.io/merchant-of-record/finance/payouts) 为准；不代表项目主体全部税务义务由平台承担 |
| 事件与权益 | 使用原始请求体验证 `creem-signature`，按事件标识幂等处理；覆盖支付、续费、取消、退款及争议，不依赖成功页参数发放权益 |
| 部署接入 | 服务端采用 `creem` SDK；Next.js 按需使用 `@creem_io/nextjs`。Workers 验证兼容性，必要时通过 REST API 或 VPS Docker 服务接入 |
| AI 图片 / 视频产品 | 按 [官方审核要求](https://docs.creem.io/features/moderation) 将每次生成接入 Moderation API，并将审核结果 ID 关联积分消费；产品审核通过后再上线收款 |
| 既有项目迁移 | 商品、客户、订阅、凭据与事件协议分别迁移；不能将原平台密钥重命名后视为完成迁移，历史退款与存量订阅需单独安排 |

### 12.2 Cloudflare 原生专用收费与协议集成

| 能力 | 提供的功能说明 | 适用业务场景 | 选用边界与套餐关系 |
| --- | --- | --- | --- |
| [Monetization Gateway](https://developers.cloudflare.com/monetization-gateway/) | 对匹配请求附加 x402 支付要求，验证授权并经支付 Facilitator 结算 | API、MCP 工具、网站和数据集的付费访问 | 当前封闭测试，买卖双方须位于美国，另有账号与域名资格要求；不是国内支付宝收款或通用商店订阅替代，见[资格说明](https://developers.cloudflare.com/monetization-gateway/eligibility/) |
| [Pay Per Crawl](https://developers.cloudflare.com/ai-crawl-control/features/pay-per-crawl/what-is-pay-per-crawl/) | 对 AI 爬虫访问内容设置价格，Cloudflare 在此场景担任 Merchant of Record | 内容站向 AI 爬虫收取访问费用 | 当前封闭测试，按账号资格开通；用途是爬虫内容访问，不扩大为普通电商或 SaaS 订阅支付 |
| [x402 协议集成](https://developers.cloudflare.com/agents/tools/payments/x402/) | 通过 HTTP 402 协商支付，由客户端签署授权，服务端验证与结算后返回资源 | Agent 程序化购买、按请求收费的工具 | 协议与集成路径，不等于 Cloudflare 经营全部支付渠道；钱包、网络和 Facilitator 另选，不作为普通国内支付默认方案 |
| [MPP 协议集成](https://developers.cloudflare.com/agents/tools/payments/mpp/) | 用统一付款挑战与凭证完成 HTTP 或 MCP 付费交互，协议定义单次、按用量及订阅意图 | 自主开发的付费 API、MCP 工具、Agent 服务 | 第三方协议与 SDK 集成，可在 Workers 运行，不是 Cloudflare 原生托管商户或订阅系统；支付方式与 SDK 逐项核对，不因协议定义 subscription 就承诺已具备全部订阅生命周期功能 |
| [AI Gateway Machine Payments](https://developers.cloudflare.com/ai-gateway/features/machine-payments/) | 通过 x402 和稳定币钱包支付指定模型的推理调用费用 | 应用或 Agent 向 Cloudflare 购买模型推理 | 这是支付模型成本，不是应用向终端客户收款；当前限美国主体、需信用卡和 API Token，仅适用官方列出的接口及模型，不推广到所有模型供应商 |

### 12.3 按实际支付场景选择

**先确认谁付款、谁收款、购买什么，再确定渠道与接入形态。** 以下为新项目的选型建议；同一项目可以既向用户收费，又向模型平台支付成本，这两笔资金流应分别设计。

| 实际业务场景 | 谁付款 → 谁收款 | 优先选择 | 使用前提与避免误选 |
| --- | --- | --- | --- |
| 国内网站购买服务、套餐、报告或点数 | 国内用户 → 项目商户 | 支付宝 + Workers + 业务数据库 | 使用已签约的适用支付产品；点数余额、有效期和消费记录由业务维护，不由支付宝或 Cloudflare 自动提供 |
| 国内会员到期后手动续费 | 国内用户 → 项目商户 | 支付宝一次性付款 + 应用会员有效期管理 | 每次续费由用户主动支付；这是会员业务，不等于自动扣款订阅 |
| 国内周期自动扣费 | 已授权用户 → 项目商户 | 支付宝适用的签约与扣款产品，另行确认开通 | 先确认商户资格、业务适用性和用户授权；普通网站支付不能自动升级为代扣，未具备条件时采用手动续费 |
| 海外网站一次性购买或自动订阅 | 海外用户 → 符合开户条件的商户 | Creem 托管收银台；订阅采用其支持的自动续费方式 | 核对商户主体、地区、币种和支付方式；订阅还需处理续费失败、取消、退款与权益同步 |
| 原生 App 内购买数字内容或订阅 | App 用户 → 适用商店或支付渠道 | 按目标商店规则选择 StoreKit / Google Play Billing，按需使用 RevenueCat | 发行地区、业务类型和渠道规则优先；国内支付宝优先原则不自动绕过商店内购要求 |
| 普通用户购买网站会员后访问内容 | 网站用户 → 项目商户 | 国内支付宝、海外评估 Creem + 登录与权益校验 | 不因内容是付费页面就直接选 Pay Per Crawl；普通用户账号、退款和会员期限仍是应用业务 |
| 已有 API / 数据集，希望由边缘网关按请求收费 | 支持付款协议的调用方 → 收款钱包 | 满足资格后评估 Monetization Gateway | 适合尽量保留原有业务接口的场景；当前封闭测试且买卖双方须在美国，接入还需后端验证付款上下文，见[接入指南](https://developers.cloudflare.com/monetization-gateway/get-started/) |
| 自己开发 Worker API / MCP 工具，由 Agent 付款调用 | Agent 或程序客户端 → 服务提供方 | 有明确 x402 客户端需求时使用 x402；需要 MPP 交互时评估 MPP SDK | 自行部署协议处理与付费路由；核对钱包、资产、网络、支付渠道与客户端兼容，不能假设普通浏览器或支付宝能直接完成机器支付 |
| 内容站向 AI 爬虫收费 | 参与付费机制的 AI 爬虫 → 内容发布者 | Pay Per Crawl | 当前封闭测试；不是所有爬虫都会支付。按[收益管理说明](https://developers.cloudflare.com/ai-crawl-control/features/pay-per-crawl/use-pay-per-crawl-as-site-owner/manage-payouts/)创建专用 Stripe Connect 账户，不直接复用已有 Stripe 账户 |
| 应用需要用钱包支付模型调用费 | 应用或 Agent → Cloudflare 模型服务 | 符合条件时评估 AI Gateway Machine Payments | 属于供应商成本支付；不能把它当作客户付款入口，也不能据此认定可用支付宝支付这些调用 |

| 容易混淆的概念 | 正确判断 |
| --- | --- |
| “有会员”与“有自动订阅” | 一次付款也能购买定期会员；只有明确的签约、周期扣款与生命周期处理才属于自动续订方案 |
| “按用量收费”与“使用机器支付协议” | 普通 SaaS 可以用支付宝购买点数，再由服务端计量扣减；不必须引入 x402 / MPP |
| “Workers 支持付款接口”与“Cloudflare 实际收款” | Workers 承载支付宝或 Creem 适配时，实际支付与结算渠道仍是外部平台 |
| “协议可扩展支付方式”与“已经支持支付宝” | MPP 可扩展不代表当前选用 SDK 已提供支付宝接入；国内普通收款仍按支付宝官方产品实施 |
| “美国资格限制”与“所有机器支付部署都受同一限制” | Monetization Gateway、AI Gateway Machine Payments 的限制按各产品适用；自部署协议方案另核对所用支付渠道与服务商，不照搬或忽略资格条件 |

新项目应在架构文档中记录所选场景、付款方、收款方、实际渠道、计价方式及权益交付规则。支付宝、Creem、网关收费和模型费用支付不能共用一个含义不明的“支付服务”配置。

## 13. 按新项目业务快速组合

以下是起步组合，数据库和身份系统可复用已有方案。只有业务需要时才增加后面的能力。

| 新项目类型 | 起步能力组合 | 随需求增加 | 需要额外解决的业务部分 |
| --- | --- | --- | --- |
| 静态官网 / 活动页 | Static Assets 或 Pages + 域名 / HTTPS | Web Analytics、Turnstile 表单验证 | 联系表单保存、通知发送和用户授权 |
| 登录后的 SaaS | Workers + 所选框架路径 + 数据库 + 身份系统 | R2、Turnstile、Logs、公共内容缓存 | 账号、租户隔离、角色权限、业务数据规则 |
| 自定义产品账号系统 | Workers + D1 + Better Auth | Turnstile、所需身份提供商、Email Sending | 认证表迁移、会话、注册登录界面、邮件验证与业务权限；验证框架、插件与邮件发送资格 |
| 邀请制 / 受控访问门户 | Access + 邮箱验证码或已接入身份提供商 + 业务应用 | Passkeys、业务数据库、Logs | Access 允许策略、用户映射和应用内资源权限 |
| 小型内容工具 | Workers + D1 | R2、公共内容缓存 | 用户授权与数据访问规则 |
| 文件管理 / 文件转换 | Workers + R2 + 数据库 | Queues / Workflows；需要系统工具时加 Containers | 文件所有权、格式校验、上传限制、删除与任务状态 |
| AI 摘要 / 写作工具 | Workers + Workers AI 或外部模型 | R2、异步任务、AI Gateway | 额度、输入输出校验、模型质量和成本控制 |
| 知识库问答 | Workers + 原文存储 + 向量生成 + Vectorize + 生成模型 | 数据导入队列、流程编排、模型调用管理 | 文档权限、检索隔离、答案质量与引用设计 |
| 多人协作 / 房间应用 | Workers + Durable Objects + 持久状态 | WebSocket、DO Alarms、业务数据库 | 身份、房间权限、重连、消息顺序和冲突处理 |
| 多人实时音视频会议 | Realtime SFU + 信令后端 + 身份系统 | TURN、DO 房间状态、业务数据库 | 音轨发布订阅、会议权限、设备管理和弱网恢复 |
| 多语言音视频会议 | Realtime SFU + WebSocket 适配器 + 翻译网关 + 百炼同传 | TURN、房间状态与配额监控 | 音频格式转换、独立音轨与语言频道、译音播放、延迟和成本验证 |
| 视频课程 / 会员内容 | Workers + Stream + 数据库 + 身份系统 | Images、Web Analytics | 购买和观看权限、支付验签、课程管理 |
| 内部管理工具 | Workers + 数据库 + Access | Logs、Cron | 后台业务角色、审计和资源授权 |
| 国内支付型应用 | Workers + 业务数据库 + 支付宝 | Queues、Issues、Email Sending、费用提醒 | 商户与支付产品开通、验签、订单金额、幂等入账、退款和权益 |
| 海外支付型应用 | Workers + 业务数据库 + Creem | Queues、Traces、Email Sending | 商户地区与渠道资格、Webhook 验签、订阅状态和权益同步 |
| 用户通知 / 邮件业务 | Workers + 订阅或业务数据 + 所需通知通道 | Agent / DO 调度、Web Push、Email Sending / Routing | 浏览器推送授权、设备通道、邮件域名、重试和失效订阅清理 |
| 业务指标 / 广告事件 | Workers + Analytics Engine；需第三方工具分发时增加 Zaraz | HTTP Events API、报表和告警 | 事件模型、采样、同意管理、数据最小化和端点防滥用 |
| 服务端错误与链路监控 | Issues + Logs + Traces | Alerts、OTel 导出 | 启用采集、Source Map、采样、保留期和告警策略；移动端诊断独立配置 |
| AI 爬虫内容变现 | 满足资格后评估 Pay Per Crawl | 爬虫访问分析、收益配置 | 面向参与付款的爬虫，封闭测试及专用结算账户；不作为普通会员支付 |
| 托管网关按请求收费 | 满足资格后评估 Monetization Gateway | 原站付款上下文验证、访问分析 | 当前买卖双方美国资格、封闭测试与钱包支付要求 |
| 自部署机器付费 API / MCP | Workers + x402 或 MPP 协议适配 | 支付验证、结算、资源交付和用量记录 | 客户端与协议、资产和渠道匹配；不是默认的国内收银方案 |
| 钱包支付模型成本 | 符合条件后评估 AI Gateway Machine Payments | 模型用量与费用监控 | 向模型服务付款，独立于向项目客户收款 |

## 14. 实施前的选型检查

| 要回答的问题 | 判断方式 |
| --- | --- |
| 是否已有可复用方案 | 已有数据库、队列、身份系统能满足需求时，先扩展已有能力 |
| 数据属于哪一种 | 结构化业务数据、文件、缓存、对象协调状态分别选择，不混用 |
| 是同步请求还是后台工作 | 用户需要立即结果时走请求；可延后、耗时或需重试时设计异步 |
| 是否依赖系统程序 | 依赖系统工具、原生二进制或完整进程环境时评估 Containers |
| 是否允许旧数据 | 能接受短暂陈旧才评估共享缓存；支付、余额等依赖可信状态 |
| 是否属于敏感资产 | 先设计授权和隔离，再考虑缓存、签名、日志或模型调用 |
| 登录与授权是否分清 | 区分 Access 门禁、Cloudflare 账号 OAuth 与自部署 Auth；验证会话、退出、账号映射和资源归属，按身份套餐或框架部署方案核算费用 |
| 通知到底发给谁 | 区分运维告警、浏览器 Web Push、原生设备推送和事务邮件，不能用一个“通知”产品承诺全部送达能力 |
| 监控覆盖哪一端 | 区分 Workers 错误与链路、网页 RUM、移动原生崩溃；统计采样不能承担精确财务记账 |
| 支付渠道是否适用 | 国内普通支付优先支付宝；验证终端、商户资格和实际产品，Cloudflare 专用收费另核对测试状态、地区与账号条件 |
| 套餐是否覆盖 | 检查产品资格、额度、资源上限、独立计费以及额外第三方账单 |
| 是否适合目标用户地区 | 对真实用户到应用、数据库、对象存储和模型的链路测延迟，不只看节点分布 |
| 实时音视频链路是否成立 | 分别验证 SFU 媒体路由、TURN 受限网络连接、适配器音频格式、模型协议、并发与译音回传，核对 [Realtime 限制](https://developers.cloudflare.com/realtime/sfu/platform/limits/) |
| 是否经过运行时验证 | 检查框架版本、兼容标记、依赖与绑定，本地测试后做隔离平台验收 |


