# 开发笔记模块 (Note Module) 实现计划

根据《Project Specification Document.md》的详细设计，笔记模块是本博客系统的核心功能，承载“学习记录” (type=0)、“生活随笔” (type=1) 和“兴趣使然” (type=2) 的内容沉淀与分类。

## Proposed Changes

### 1. 数据库脚本 (SQL)
#### [NEW] [note_module.sql](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/sql/note_module.sql)
新增三张表：
- `blog_note` (笔记主表): 存储文章基础信息及 Markdown 长文本 (`MEDIUMTEXT`)。
- `blog_tag` (标签主表): 存储全局唯一的标签名称。
- `blog_note_tag` (关联表): 维护文章与标签的多对多关系。

---

### 2. 后端服务 (Backend - Spring Boot / Java 17)
#### [NEW] Entities
- `src/main/java/com/czf/blog/entity/BlogNote.java`
- `src/main/java/com/czf/blog/entity/BlogTag.java`
- `src/main/java/com/czf/blog/entity/BlogNoteTag.java`

#### [NEW] DTOs & VOs
- `src/main/java/com/czf/blog/dto/NoteSaveDTO.java` (入参，包含标签列表)
- `src/main/java/com/czf/blog/dto/NoteListVO.java` (出参，不包含 Markdown 全文用于分页)
- `src/main/java/com/czf/blog/dto/NoteDetailVO.java` (出参，包含全文及关联的标签集合)

#### [NEW] Mappers
- 定义 MyBatis Plus 的 `BaseMapper` 接口，以便支持快速的 CRUD：
  - `BlogNoteMapper`
  - `BlogTagMapper`
  - `BlogNoteTagMapper`

#### [NEW] Service & Impl
- `NoteService` & `NoteServiceImpl`:
  - `saveOrUpdateNote`: 包含全文本处理、摘要自动截取（前 150 字），处理关联标签（先删后增，并保证全局标签唯一性）。
  - `getNotePage`: 分页查询对应类型的笔记列表（返回 VO）。
  - `getNoteDetail`: 根据 ID 查询全文，并异步或者立刻 `UPDATE views = views + 1` 增加访问量。

#### [NEW] Controller
- `src/main/java/com/czf/blog/controller/NoteController.java` 
暴露对应的 `/api/note/save`, `/api/note/list`, `/api/note/{id}` 等接口。

---

### 3. 前端服务 (Frontend - Vue3)
#### [NEW] [note.ts](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-frontend/src/api/note.ts)
新增文章相关的 Axios API 接口调用封装 (`noteApi`)。

#### [NEW] [WriteView.vue](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-frontend/src/views/WriteView.vue)
- 接入 `md-editor-v3` Markdown 编辑器。
- 支持输入标题、选择模块类型（学习/随笔/兴趣），以及编辑文章的 tags 列表。
- 集成保存草稿和发布功能，调用后端接口。

---

## Verification Plan

### Automated Tests
- 执行 `mvn test` 确认项目编译正常，不破坏原有内容。
- 在前端使用 `npm run dev` 运行开发服务器。

### Manual Verification
1. 启动后端数据库，运行 `note_module.sql` 初始化表结构。
2. 启动 Spring Boot 后端项目。
3. 启动 Vite 前端并访问首页，点击“写笔记”进入 `WriteView`。
4. 输入标题、Markdown 内容、选择类型，点击“发布”。
5. 调用查询列表接口，验证内容和标签能正确查出。
6. 点击文章详情，验证阅读量 `views` 成功增加 1。
