#  Project Specification Document

## 项目的目标 (What is the goal?)

本项目旨在构建一个**高度集成、私有化的个人管理与内容沉淀中心**，也就是个人的博客管理系统，开发者能够随心添加自己喜欢的功能。它不仅是一个对外展示的博客，更是一个对内的“数字化人生仪表盘''。供开发者个人使用。

当前的目标：

* **多维度的自我量化**：通过追番、打卡等模块，将虚无的时间消耗转化为可见的数据指标。
* **结构化的知识管理**：明确区分“职业学习”与“兴趣探索”，建立清晰的个人知识边界，实现程序员的技术积淀。
* **情感与记忆的留存**：通过随笔捕捉日常生活点滴，形成可回溯的个人历史时间轴。
* **习惯的视觉反馈**：利用多类型打卡系统，为日常琐事（如学习、健身、早起）提供正向的反馈机制。

## 1 Project Requirement Doc

### 项目的功能：

本项目采用 **MVP (Minimum Viable Product)** 迭代模式，优先跑通核心业务流，再逐步完善自动化与视觉体验。

### **版本划分表**

| 阶段         | 版本           | 核心关注点             | 核心功能                                                     | 交付标准                                                     |
| :----------- | :------------- | :--------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| **第一阶段** | **V1.0 (MVP)** | **基础框架与记录功能** | **首页**：<br>展示学习记录，生活随笔，追番管理的跳转标签，一个时间、一个日历。首页界面参考https://lvyovo-wiki.tech<br>2. 学习记录（Markdown 支持，可上传图片）<br>3. 生活随笔（Matkdown支持，基础图文）<br>3. 追番管理基础版（对接 Bangumi API） | 实现基础内容 CRUD，能正常发布文章，并能导入通过bangumi的api导入正在看的番剧，并能勾选已经看过的番剧的集数 |
| **第二阶段** | **V2.0**       | **外部集成与交互优化** | 1. 追番增强版<br>2. 每日打卡基础版（单任务打卡）<br>3. 兴趣使然模块（非主业学习记录） | 并能开始简单的每日任务打卡，实现番剧的按季度分类、按追番时间查看 |
| **第三阶段** | **V3.0 **      | **数据可视化与全模块** | 1. 每日打卡（多类型、日历/热力图视图）<br>2. 全局数据统计看板<br>3. 全站搜索与标签系统 | 实现多任务并行打卡，通过首页看板直观展示学习时长、追番进度及打卡趋势。 |

## 2 Engineering Doc

### 2. 1 技术栈

### 2.2 系统架构

项目采用**前后端分离的单体架构**。前端负责路由控制与 UI 交互，后端负责业务逻辑持久化与第三方服务对接。

#### 1. 前端（Presentation Layer）

- **用户 Web 端**: 使用 **Vue 3 + Tailwind CSS** 构建。
- **管理后台**: 直接集成在 Web 端中（通过角色权限控制），实现文章发布、番剧导入。

#### 2. 接入层（Infrastructure & Gateway）

- **Nginx**: 负责静态资源托管和请求转发，处理 HTTPS 证书。
- **统一结果封装**: 对应图中网关的鉴权逻辑，我们在后端代码中通过 `Interceptor`（拦截器）实现简单的 Token 校验。

#### 3. 核心服务模块（Service Modules）

在单体架构中，这些是后端项目内部的**业务包（Package）**，而非独立进程：

- **内容服务 (Content Service)**: 负责“学习记录”、“生活随笔”、“兴趣使然”的逻辑。
- **追番服务 (Anime Service)**: 负责对接 **Bangumi API** 和进度管理。
- **打卡服务 (Check-in Service)**: 处理每日任务逻辑（V2.0 重点）。
- **文件服务 (File Service)**: 专门封装对 **MinIO** 的操作。

#### 4. 数据存储层（Data Storage）

- **关系型数据库 (MySQL 8.0)**: 存储所有核心业务数据（文本使用 `MEDIUMTEXT`，进度使用 `JSON`）。
- **Redis 缓存**: 用于缓存 Bangumi API 数据，防止频繁请求被封 IP。
- **对象存储 (MinIO)**: 存储 Markdown 中的图片、番剧封面图。

#### 5. DevOps

- **Git**: 源代码管理。
- **Docker**: 将 MySQL、MinIO、Redis 和你的后端程序全部容器化。

#### Project Structure

```text
PersonalBlog/
├── PersonalBlog-backend/         # Spring Boot 3 + Java 17 后端项目
│   ├── sql/                      # 数据库初始化脚本
│   ├── src/main/java/com/czf/blog/
│   │   ├── common/               # 全局公共类（统一返回结果、异常处理、常量等）
│   │   ├── config/               # 核心配置类（MyBatis Plus、MVC等）
│   │   ├── controller/           # 接口层 (Controller)
│   │   ├── dto/                  # 数据传输对象 (Request/Response models)
│   │   ├── entity/               # 数据库实体类
│   │   ├── mapper/               # 数据访问层 (MyBatis Mapper)
│   │   └── service/              # 业务逻辑层 (包括 Content, Anime, Check-in 等内部服务)
│   └── src/main/resources/
│       ├── application.yml       # 核心配置文件
│       └── mapper/               # MyBatis XML 映射文件
├── PersonalBlog-frontend/        # Vue 3 + Vite 前端项目
│   ├── public/                   # 静态资源
│   ├── src/
│   │   ├── api/                  # Axios 接口请求封装
│   │   ├── components/           # UI 公共组件
│   │   ├── utils/                # 工具类封装
│   │   ├── views/                # 页面级组件 (如 Home, Anime, CheckIn等)
│   │   ├── App.vue               # Vue 根组件
│   │   └── main.ts               # 前端项目入口文件
│   ├── package.json              # 依赖管理
│   └── vite.config.ts            # Vite 构建设定
└── ProjectDocument/              # 项目文档存放处
    └── Project Specification Document.md # 需求与设计说明书
```
### 2.3 系统设计

#### 2.3.1 概要设计
##### 模块划分
    * 内容服务 (Content Service): 负责“学习记录”、“生活随笔”、“兴趣使然”的逻辑。
    * 追番服务 (Anime Service): 负责对接 **Bangumi API** 和进度管理。
    * 打卡服务 (Check-in Service): 处理每日任务逻辑（V2.0 重点）。
    * 文件服务 (File Service): 专门封装对 **MinIO** 的操作。
##### 技术栈选型(MVP V1.0)：

本项目采用前后端分离的单体架构，选型侧重于“开发效率”与“数据灵活性”的平衡。

**后端 (Backend)**
* **核心框架**: `JDK 17` + `Spring Boot 3.3+`
* **持久层**: `MyBatis Plus`
* **数据库**: `MySQL 8.0`
* **对象存储**: `MinIO`
* **辅助依赖**:
    * `Lombok`: 消除样板代码。
    * `Spring Validation`: 后端接口入参自动化校验。

**前端 (Frontend)**
* **核心框架**: `Vue 3 (Composition API)` + `Vite`
* **UI 组件库**: `Element Plus`
* **样式框架**: `Tailwind CSS`
* **状态管理**: `Pinia`
* **核心插件**:
    * `Axios`: 异步请求封装。
    * `md-editor-v3`: 专业的 Markdown 编辑器，支持代码高亮与图片上传接口。
    * `@tailwindcss/typography`: 官方排版插件，一键美化 Markdown 渲染效果。

#### 2.3.2 详细设计

