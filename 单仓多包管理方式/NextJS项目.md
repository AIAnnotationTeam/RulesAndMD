# （单仓多包）管理方式
采用的（单仓多包）管理工具为：Turborepo

项目目录结构 (Turbo + Next.js)
repready-web/
├── turbo.json              # Turbo 任务配置文件
├── package.json            # 根目录依赖
├── apps/                   # 应用层
│   ├── admin/              # Next.js - 团队管理管理人员 Web App
│   ├── user/               # Next.js - 销售代表 Web App (含 WebRTC)
│   └── server/             # 服务
├── packages/               # 共享库层
│   ├── ui/                 # 共享 React 组件库 (基于 Figma 设计)
│   ├── firebase/           # 共享的 Firebase Admin/Client 初始化逻辑
│   ├── openai/             # 封装的 OpenAI 的 WebRTC Session Token 和 RAG 逻辑
│   ├── typescript-config/  # 统一的 tsconfig 配置
│   └── eslint-config/      # 统一的代码规范
└── firebase.json           # Firebase 部署配置
