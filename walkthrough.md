# 笔记模块 (Note Module) 交付总结

根据系统规范，本项目已实现基础的「笔记发布与沉淀」功能，包括后端的数据支撑以及一套美观实用的富文本/Markdown前台界面。

## 做了哪些改动 (Changes Made)

### 1. 数据库结构升级 (Database)
- 初始化了核心笔记三表架构：
  - `blog_note`: 存储笔记的正文(MEDIUMTEXT)、标题、摘要、阅览量，以及对应所属板块（0:学习, 1:随笔, 2:兴趣）。
  - `blog_tag`: 全局标签库。
  - `blog_note_tag`: 笔记与标签的多对多对应表。

### 2. 后端服务开发 (Backend - `com.czf.blog`)
- **Entity & DTO** 层：创建了 [BlogNote](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/entity/BlogNote.java)、[BlogTag](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/entity/BlogTag.java) 及针对保存和分页查询优化的 [NoteSaveDTO](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/dto/NoteSaveDTO.java), [NoteListVO](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/dto/NoteListVO.java), [NoteDetailVO](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/dto/NoteDetailVO.java)。
- **Mapper** 层：集成 *MyBatis Plus*，完成基础 DAO 接入。
- **Service** 层 ([NoteServiceImpl](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/service/impl/NoteServiceImpl.java))：
  - [saveOrUpdateNote](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/service/impl/NoteServiceImpl.java)：支持笔记持久化。如果未提供摘要，**会自动提取脱敏后的 Markdown 纯文本截取 150 字补充**。同时处理并维护新增的 Tags。
  - 对用户意见进行了针对性加强：增强了对 `empty tag collection` 时的 MyBatis 健壮性检查。
  - [getNotePage](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/service/impl/NoteServiceImpl.java) 与 [getNoteDetail](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/service/impl/NoteServiceImpl.java)：实现基础阅读查询。
- **Controller** 层 ([NoteController](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-backend/src/main/java/com/czf/blog/controller/NoteController.java))：提供 `/api/note/save`、`/api/note/list`、`/api/note/{id}`。已通过 `mvn clean compile` 编译通过。

### 3. 前端交互与视图 (Frontend - Vue3)
- 引入最新版 `md-editor-v3` 依赖作为轻量级与功能全面的 Markdown 书写组件。
- 新增：[src/api/note.ts](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-frontend/src/api/note.ts) 类型声明与 REST 请求绑定。
- 新增核心业务界面：[WriteView.vue](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-frontend/src/views/WriteView.vue)。采用高度统一的**绿白渐变毛玻璃 (Bento Grid) 美学**设计。
  - 侧边栏支持便捷录入：笔记板块选择、动态标签生成（输入后回车）、封面图直链预览配置。
  - 主副区支持 `md-editor-v3` 全屏无边框编辑。
  - [main.ts](file:///c:/DevelopmentTool/Code/PersonalBlog/PersonalBlog-frontend/src/main.ts) 已支持路由映射至 `/write`。

---

## 验收建议 (Manual Verification)

为确认整个应用链路运转顺利，请执行以下验证：

1. 如果尚未重启后端，请在后端项目根目录运行 `mvn spring-boot:run` 重新启动服务，以加载最新的 Controller 和 Mapper。
2. Frontend dev server 应该正在运行并自动热更新。如果在浏览器中打开 `http://localhost:5174/write`。
3. 随意键入标题和正文，加上自定义标签，并在分类中选择一个模块进行测试。
4. 点击右上方**「发布」**按钮。由于系统内部集成了 `axios` 与 `url proxy`，前端的 API 请求应通过网关最终落入你本地刚才创建的 `blog_note` 表中。
