# Figma 交付说明

| 项目 | 内容 |
| --- | --- |
| 设计文件 | [同言 Tongyan · 实时多语言会议 UI·UX](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL) |
| 所属团队 | superMember，按周先生选择创建 |
| 界面页面 | 01 · 桌面界面 |
| 组件页面 | 02 · 参会者组件 |
| 设计方式 | 原生文本、自动布局、变量绑定、组件实例；非整页截图粘贴 |
| 可复用组件 | Simple Design System Button；本地 Participant tile |
| 文字 | 本次画板中文均为 Noto Sans SC，已读取实际字体核对 |
| 基础样式 | Tongyan 配色、间距变量和文字样式 |
| 原型方式 | 核心画板跳转；完整交互逻辑在本地 HTML 原型 |

| 画板 | Figma 直达链接 | 内容 |
| --- | --- | --- |
| 工作台 | [01 工作台](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=3-26) | 即时会议、链接加入、未开始会议 |
| 会前准备 | [02 会前准备](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=3-27) | 设备关闭、昵称、语言、模式、告知 |
| 默认会议 | [03 会议室 · 语言频道](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=3-28) | 视频宫格、字幕、个人语言面板、底栏 |
| 协作会议 | [04 共享屏幕 · 聊天](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=3-29) | 共享舞台、成员缩略区、字幕、聊天 |
| 等待 | [05 等待加入](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=3-30) | 等待审批、取消、隐私说明 |
| 结束 | [06 会议结束](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=3-31) | 结束说明、返回、新建 |
| 十人会议 | [07 十人会议](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=19-71) | 十人成员、自适应紧凑卡片、持续字幕与语言频道 |
| 紧凑成员组件 | [Participant tile / Compact](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=21-127) | 十人会议复用实例 |
| 参会者组件 | [Participant tile](https://www.figma.com/design/XZZbVyn3v7Ufk4K3NNSRLL?node-id=5-6) | 姓名与语言文本属性、复用实例 |

| 检查 | 结果 |
| --- | --- |
| 画板范围 | 所列核心画板均已填充 |
| 容器边界 | 最终读回未发现子节点超出父容器边界 |
| 多行文本 | 已修复共享标题、结束说明等高度引发的视觉重叠 |
| 深色辅助文字 | 独立使用更亮的舞台文字变量 |
| 主要跳转 | 工作台 → 准备 → 会议；会议 ↔ 共享；等待 → 准备；结束 → 工作台 / 新建 |
| 设计边界 | Figma 是布局与组件设计稿，HTML 包含更多弹窗、审批和异常状态；不是生产实现 |
| 本地存档 | 本目录保存原型、截图、规范、变量和 Figma 定位信息，不包含原生 .fig 文件 |
| 人数修订 | 最大人数改为 10 人（含主持人）；新增十人画板与紧凑组件，截图核对及边界检查通过 |
| 语言文案修订 | 会前与会中统一为“将其他人的发言翻译成”；已补充多种源语言统一译为中文及中文保留原声的说明 |

网络中断期间已核对服务端实际保存状态，避免将未返回的请求直接判定为未提交；最终以画板读回与边界检查结果为准。
