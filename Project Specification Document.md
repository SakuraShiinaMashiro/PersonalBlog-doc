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
| **第一阶段** | **V1.0 (MVP)** | **基础框架与记录功能** | **首页 (Home)**：<br>1. **极简视觉风格**: 参考 lvyovo-wiki，采用磨砂玻璃卡片布局。<br>2. **动态挂件**: 集成实时数字时钟（带问候语）与极简日历。<br>3. **快速导航卡片**: 定义“生活随笔”、“学习记录”、“兴趣使然”、“追番进度”四个核心入口。<br>4. **内容 CRUD**: 基础文章发布与 Bangumi 番剧导入及进度勾选功能。 | 实现基础内容 CRUD，首页能通过时钟与日历建立“数字化人生”的初始感。 |
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
- **安全性说明**: **V1.0 (MVP) 版本暂不实现鉴权与权限控制逻辑**。系统默认在受信任的私有网络环境下运行，所有 API 接口均为公开访问状态。鉴权模块计划在 V2.0 版本引入。

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
│   │   ├── exception/            # 自定义异常类
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

本项目采用前后端分离的单体架构，选型侧重于“开发效率”、“数据灵活性”与“极致的 UI 自定义”。

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
* **样式框架**: `Tailwind CSS` (作为唯一的 UI 基础)
* **状态管理**: `Pinia`
* **核心插件**:
    * `Axios`: 异步请求封装。
    * `md-editor-v3`: 专业的 Markdown 编辑器，支持代码高亮与图片上传接口。
    * `@tailwindcss/typography`: 官方排版插件，一键美化 Markdown 渲染效果。
    * `Lucide Vue Next`: 轻量级图标库。

#### 2.3.2 详细设计 (Detailed Design)

本章节详细说明 V1.0 (MVP) 核心模块的工程实现方案。

---

##### 1. 公共模块与基础设施 (Infrastructure)

###### 1.1 统一响应协议 (`Result<T>`)
所有 Controller 层方法必须返回 `Result<T>`，禁止直接返回业务对象。
```java
public record Result<T>(
    int code,       // 200:成功, 400:业务错, 500:系统错
    String msg,     // 提示消息
    T data          // 业务负载
) {}

// 分页数据标准封装
public record PageData<T>(
    List<T> list,    // 数据列表
    long total,      // 总记录数
    int pageNum,     // 当前页码
    int pageSize     // 每页大小
) {}
```

###### 1.2 全局异常处理逻辑
*   **业务异常 (`BusinessException`)**: 继承 `RuntimeException`，由 `GlobalExceptionHandler` 统一捕获并封装为 `Result.error(code, msg)`。
*   **参数校验**: 配合 `@Valid` 注解，捕获 `MethodArgumentNotValidException`，返回 400 状态码及具体错误字段。

###### 1.3 文件存储 (File Service)
*   **存储引擎**: MinIO (本地磁盘映射模式)。
*   **桶规范**: `blog-assets`
*   **命名策略**: `module/yyyyMMdd/UUID_originalName.ext` (如 `article/20240318/a1b2_demo.png`)。
*   **清理机制**: MVP 阶段暂不实现物理删除，采用数据库记录与文件系统解耦。

---

##### 2. 首页聚合模块 (Home/Dashboard Module)

###### 2.1 模块职责与设计目标
首页作为系统的“大脑”，旨在通过单一请求提供全站状态感知。设计遵循“一眼洞察”原则，减少用户在不同页面间频繁跳转来确认进度的操作。

###### 2.2 具体组件功能
1.  **数字时钟挂件 (Digital Clock)**:
    *   **功能**: 显示实时 24 小时制时间。
    *   **实现**: 前端使用 `ref` 响应式变量，配合 `requestAnimationFrame` 或 `setInterval` 实现平滑跳动，不产生后端负载。
2.  **极简交互日历 (Mini Calendar)**:
    *   **功能**: 展示当月日期。
    *   **打卡感知**: 接收后端返回的日期字符串数组（如 `["2024-03-01", "2024-03-18"]`），在对应日期下方显示微小的色点（Dot Indicator），提示该日有内容产出（学习记录、随笔或追番更新）。
3.  **模块统计卡片 (Statistic Glass-Cards)**:
    *   **功能**: 实时汇总核心指标。
    *   **统计口径**:
        *   `学习记录`: 数据库中 `module_type = 0` 的文章总数。
        *   `生活随笔`: 数据库中 `module_type = 1` 的文章总数。
        *   `兴趣使然`: 数据库中 `module_type = 2` 的文章总数。
        *   `正在追番`: `blog_anime_progress` 中状态为“在看”且未看完所有集数的番剧数量。
4.  **最近动态馈送 (Activity Feed)**:
    *   **功能**: 混合展示全站最新的 5 条动态。
    *   **数据聚合逻辑**: 通过 `UNION ALL` 或分表查询，提取文章创建记录与番剧进度更新记录，按时间倒序排列。

###### 2.3 聚合接口设计细节 (`GET /api/home/dashboard`)
后端 `HomeService` 负责编排应用内部各业务 Service 组件的数据抓取。虽然是单体架构，但为了优化数据库查询性能（减少串行等待），建议在 `HomeService` 内部采用多线程并行查询。
*   **输入**: 无需入参（或可选年份/月份用于日历标记）。
*   **数据聚合逻辑**:
    1. 调用 `NoteService.getCountsGroupByModule()`：从 `blog_note` 表按模块类型统计数量。
    2. 调用 `AnimeService.getWatchingCount()`：从 `blog_anime_progress` 统计“在看”番剧。
    3. 调用 `ActivityService.getLatestActivities(5)`：聚合笔记与进度的最新时间戳记录。
    4. 构造 `DashboardVO` 并返回。
*   **性能建议**: 即使在单体中，也可使用 `CompletableFuture.supplyAsync()` 来并发执行上述 3 个查询任务，最后由 `join()` 汇总结果，显著降低接口 RT（响应时间）。

---

##### 3. 笔记模块 (Note Module)

###### 3.1 扩展数据库 Schema 设计
笔记模块采用多表关联方案，以支持结构化知识管理。

**A. 笔记主表 (`blog_note`)**
| 字段名 | 类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PK, AUTO` | 主键 (分布式 ID 或自增) |
| `title` | `VARCHAR(200)` | `NOT NULL` | 标题 |
| `summary` | `VARCHAR(500)` | `NULL` | 摘要 (列表页展示，若为空则后端截取原文) |
| `content` | `MEDIUMTEXT` | `NOT NULL` | Markdown 原文 |
| `module_type` | `TINYINT` | `INDEX` | 0:学习, 1:随笔, 2:兴趣使然 |
| `status` | `TINYINT` | `DEFAULT 1` | 0:草稿, 1:发布 |
| `cover_url` | `VARCHAR(500)` | `NULL` | 封面图 URL |
| `views` | `INT` | `DEFAULT 0` | 阅读量统计 |
| `create_time` | `DATETIME` | `DEFAULT NOW` | 创建时间 |
| `update_time` | `DATETIME` | `DEFAULT NOW` | 最后更新时间 |

**B. 标签主表 (`blog_tag`)**
| 字段名 | 类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PK, AUTO` | 标签 ID |
| `name` | `VARCHAR(50)` | `UNIQUE` | 标签名称 (如: Java, 摄影) |

**C. 笔记-标签关联表 (`blog_note_tag`)**
| 字段名 | 类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `note_id` | `BIGINT` | `INDEX` | 笔记 ID |
| `tag_id` | `BIGINT` | `INDEX` | 标签 ID |

###### 3.2 核心 API 规范
*   **保存/更新笔记 (`POST /api/note/save`)**:
    *   **幂等逻辑**: 根据入参 `id` 是否为空判断 `Insert` 或 `Update`。
    *   **标签处理**: 自动维护 `blog_tag` 的唯一性，并增量更新关联表（先删后增）。
*   **分页列表查询 (`GET /api/note/list`)**:
    *   **请求参数**: `pageNum` (默认1), `pageSize` (默认10), `module_type`, `tagId`, `keyword`。
    *   **性能规范**: 该接口**严禁返回 `content` 字段**。返回数据负载为 `PageData<NoteListVO>`。
*   **详情获取 (`GET /api/note/{id}`)**:
    *   **响应内容**: 返回完整 Markdown 及关联的标签列表。
    *   **统计更新**: 接口触发时异步增加 `views` 计数。

###### 3.3 关键实现细节
*   **Markdown 图片处理流程**:
    1. 前端 `md-editor-v3` 拦截粘贴/上传动作，调用 `File Service` 上传接口。
    2. 后端存储至 MinIO，返回公网持久化 URL。
    3. 编辑器自动将本地临时路径替换为远程 URL，并随正文保存至数据库。
*   **摘要自动生成策略**: 若前端未提供 `summary`，后端在保存前利用正则表达式剥离 Markdown 标签，截取前 150 字符作为纯文本摘要。
*   **首页数据供给**: 首页的 `statistics` 统计由各 `module_type` 的 `COUNT(*)` 聚合而成；`recentActivities` 取自 `create_time` 倒序的 Top 5 记录。

---

##### 4. 追番模块 (Anime Module)

###### 4.1 功能需求描述 (Functional Requirements)
追番模块旨在为用户提供一个闭环的动画进度管理体验，分为 **元数据获取**、**进度追踪** 和 **状态管理** 三个核心维度：
*   **远程搜索与导入 (Search & Import)**：通过后端代理请求 Bangumi API v0 搜索番剧。点击导入后，自动抓取 `标题`、`封面图`、`总集数`、`放送日期` 等元数据持久化至本地数据库，确保数据稳定性。
*   **进度追踪 (Episode Tracking)**：支持对每一集进行“已看/未看”状态的切换（幂等操作）。系统根据“已看集数 / 总集数”实时计算百分比进度并提供视觉反馈。
*   **状态与分类筛选 (Status & Filtering)**：支持 `想看 (Wish)`、`在看 (Doing)`、`已完结 (Collect)` 三种状态切换。支持按 `年份` 和 `季度 (1, 4, 7, 10月)` 进行维度过滤，方便回顾。

###### 4.2 数据库设计 (Database Design)

**表 4.1：番剧元数据表 (`blog_anime_subject`)**
| 字段名 | 类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PK, AUTO` | 内部唯一标识 |
| `bgm_id` | `INT` | `UNIQUE` | Bangumi Subject ID |
| `title` | `VARCHAR(255)` | `NOT NULL` | 番剧中文名/原名 |
| `image_url` | `VARCHAR(500)` | `NULL` | 封面图链接 |
| `eps` | `INT` | `DEFAULT 0` | 总集数 |
| `air_date` | `DATE` | `NULL` | 放送日期 |
| `air_year` | `INT` | `INDEX` | 放送年份 (用于筛选) |
| `air_season` | `TINYINT` | `INDEX` | 放送季度 (1:冬, 2:春, 3:夏, 4:秋) |
| `status` | `TINYINT` | `DEFAULT 0` | 0:想看, 1:在看, 2:已完结 |
| `create_time` | `DATETIME` | `DEFAULT NOW` | 记录创建时间 |
| `update_time` | `DATETIME` | `DEFAULT NOW` | 最后状态更新时间 |

**表 4.2：追番进度表 (`blog_anime_progress`)**
| 字段名 | 类型 | 约束 | 描述 |
| :--- | :--- | :--- | :--- |
| `id` | `BIGINT` | `PK, AUTO` | 主键 |
| `anime_id` | `BIGINT` | `UNIQUE, FK` | 关联 `blog_anime_subject.id` |
| `watched_eps` | `JSON` | `NOT NULL` | 已看集数列表，存储如 `[1, 2, 5]` |
| `update_time` | `DATETIME` | `DEFAULT NOW` | 最后进度更新时间 |

###### 4.3 API 接口设计 (API Design)

**1. 搜索番剧 (远程)**
*   **Endpoint**: `GET /api/anime/search`
*   **Request Params**: `keyword` (String)
*   **Response**: `Result<List<BangumiSubjectDTO>>`
```json
{
  "code": 200,
  "data": [
    {
      "id": 12345,
      "name_cn": "进击的巨人",
      "images": { "large": "..." },
      "eps": 24,
      "date": "2013-04-06"
    }
  ]
}
```

**2. 导入番剧**
*   **Endpoint**: `POST /api/anime/import`
*   **Request Body**: `{ "bgmId": 12345, "airYear": 2024, "airSeason": 1 }`
*   **Response**: `Result<Void>`

**3. 获取番剧列表 (带进度)**
*   **Endpoint**: `GET /api/anime/list`
*   **Request Params**: `year` (Int, optional), `season` (Int, optional)
*   **Response**: `Result<List<AnimeListItemVO>>`
```json
{
  "code": 200,
  "data": [
    {
      "subject": { "id": 1, "title": "...", "status": 1, "eps": 12 },
      "progress": { "watchedEps": [1, 2, 3] }
    }
  ]
}
```

**4. 进度与状态更新**
*   **切换单集进度**: `POST /api/anime/toggle` | 入参: `{ "animeId": 1, "episodeIndex": 5 }`
*   **更新番剧状态**: `PUT /api/anime/status` | 入参: `{ "animeId": 1, "status": 2 }`

###### 4.4 样式参考与实现 (yuc.wiki)
*   **核心布局**：参考 `yuc.wiki` 的网格化布局，使用 Tailwind 的 `grid-cols-1 md:grid-cols-3 lg:grid-cols-4`。
*   **视觉特征**：
    *   **Glassmorphism**：卡片使用 `bg-white/60 backdrop-blur-md border border-white/20`。
    *   **进度反馈**：卡片底部嵌入 `h-1` 的蓝色进度条；使用状态色块（如左侧状态条）区分“想看/在看/已完结”。
    *   **交互**：悬浮时显示集数快捷勾选面板，配合 `group-hover` 实现平滑的视觉过渡。


---

##### 5. 前端架构细节
*   **Glassmorphism 实现**: 统一 CSS 类 `.glass-card { @apply bg-white/40 border border-white/20 backdrop-blur-md; }`。
*   **数据预取**: 首页入口使用 `Suspense` 或 `onMounted` 触发聚合接口，配合全局 `Loading` 状态。
*   **Markdown 渲染**: 开启 `md-editor-v3` 的骨架屏模式，提升长文章加载体验。



