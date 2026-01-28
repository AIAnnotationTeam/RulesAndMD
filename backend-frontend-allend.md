## pure_frontend(纯前端项目)，**pure_backend(纯后端项目)，full_stack(全栈项目)** 的生成约定

## 项目统一目录结构

```
Project_Root/
├── README.md           <-- [必填] 项目说明书
├── docker-compose.yml  <-- [必填] 核心启动文件
├── frontend/           <-- 前端代码 (含 Dockerfile)
├── backend/            <-- 后端代码 (含 Dockerfile)
└── ... (其他工程文件)

```

## 项目Docker使用规则

- 所有运行依赖必须通过 Docker / docker-compose 显式声明
- 不得依赖本地服务、私有资源或公司内网
- 若包含数据库或中间件，必须一并容器化

**full_stack(全栈项目)：必须把前端、后端、数据库全部放在 Docker 里（All-in-one）**。

**pure_backend(纯后端项目)：必须把后端、数据库全部放在 Docker 里（All-in-one）**。

**pure_frontend(纯前端项目)：必须把前端全部放在 Docker 里（All-in-one）**。

### **full_stack(全栈项目)的Docker详细使用方式**

- 必须使用 `docker-compose` 编排。
- **Frontend**：必须映射端口（如 3000），且配置好 Nginx 。
- **Backend**：必须在容器内完成依赖安装（`pip install` / `mvn package` 等）。
- **Database**：如果使用了 MySQL/Redis，必须在 compose 文件中定义对应的 Service，**严禁使用连接本地 localhost 数据库的配置**。

### **pure_backend(纯后端项目)的Docker详细使用方式**

- 必须使用 `docker-compose` 编排。
- **Backend**：必须在容器内完成依赖安装（`pip install` / `mvn package` 等）。
- **Database**：如果使用了 MySQL/Redis，必须在 compose 文件中定义对应的 Service，**严禁使用连接本地 localhost 数据库的配置**。

### **pure_frontend(纯前端项目)的Docker详细使用方式**

- 必须使用 `docker-compose` 编排。
- **Frontend**：必须映射端口（如 3000），且配置好 Nginx 。

### **使用docker要实现的目标**

QA在终端 输入 `docker-compose up` -> 然后在浏览器打开 `localhost:3000` -> 验收通过。

### 标准的 docker-compose.yml 结构

`docker-compose.yml` 应该至少包含 3 个服务：

```
version: '3.8'

services:
  # 1. 数据库服务 (Database)
  db:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3306:3306"

  # 2. 后端服务 (Backend)
  backend:
    build: ./backend  # 后端代码目录
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      DB_HOST: db     # 关键：连接数据库必须用服务名，不能用 localhost

  # 3. 前端服务 (Frontend) - 必须要有这个！
  frontend:
    build: ./frontend # 前端代码目录
    ports:
      - "3000:3000"   # 映射端口供 QA 访问
    depends_on:
      - backend
    # 可以在这里定义开发命令，或者在 Dockerfile 里写好
    command: npm start

```

### 

## 使用Docker要达到的目标

**1.启动简单**

只需要 `docker compose up` 一条指令就可以完成启动，并且启动时

- 不需要额外参数
- 不需要手动修改配置或环境变量
- 不依赖本地语言运行环境

**2.项目验收简单**

在项目验收时，只需要通过 `docker compose up` 启动就可以了。

- 启动过程无需人工干预
- 验证者无需阅读源码即可完成基本验证
- 总结就一句话，需要与github高星项目一致，开箱即用

## 项目参考示例

🌟 案例：全栈 - 个人财务管理看板 (Personal Finance Dashboard)

1. 视觉与交互 (The Look)
- **UI 审美**：使用了 **Tailwind CSS**，卡片式设计，有阴影、圆角，配色舒适（不是原生 HTML 的 Times New Roman 字体）。
- **交互**：按钮有 Hover 效果，加载数据时有 Loading 转圈，操作成功有 Toast 提示。
1. 工程结构 (The Code)

```
finance-app/
├── docker-compose.yml      <-- 🟢 满分点：根目录清晰
├── README.md               <-- 🟢 满分点：文档详尽
├── frontend/
│   ├── Dockerfile
│   ├── src/
│   │   ├── components/     <-- 🟢 满分点：组件化拆分
│   │   └── pages/
│   └── package.json
└── backend/
    ├── Dockerfile
    ├── app/
    │   ├── models.py       <-- 🟢 满分点：MVC 分层
    │   └── main.py
    └── requirements.txt

```


## 技术方案选择原则
在项目里所有核心流程和功能点，必须使用当前业务场景的主流解决方案**gitHubs上有第三方库（Star超过500星）优先使用第三方库，尽量不要从0实现**。
**提示词中明确要求使用的技术点除外**。

## 数据存储方式要求
项目中的数据全部存储到数据库中，浏览器或客户端只能保存临时变量。

## 方案要求
要求方案要确定，不要出现“或”这样不确定的方案

## 当前开发环境
电脑系统：Mac OS
