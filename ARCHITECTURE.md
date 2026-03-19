# 问卷小筑 · 项目结构导航

> 本文件只用于代码导航和定位入口，不替代源码核验。
> 写 `devDoc` 文档、做现状判断、写简历结论前，仍然必须回到真实代码阅读。

---

## 1. 先看什么

推荐的阅读顺序：

1. 先看本文件，确认 monorepo 包职责和功能入口
2. 再看 `devDoc/问卷项目_前端暑期实习简历功能梳理.md`，确认当前回顾功能编号
3. 进入对应 package 的入口文件、页面、路由、controller、service
4. 最后再补 utils、slice、hooks、spec 等支撑文件

---

## 2. 仓库总览

```text
questionnaire/
├─ AGENTS.md                     # 回顾规则和知识库约束
├─ ARCHITECTURE.md               # 代码导航入口（本文件）
├─ devDoc/                       # 任务看板、四文档模板、简历沉淀
├─ doc/                          # 架构图、接口文档、补充资料（因为新做了改动，所以该文件夹下有的过时了）
├─ packages/
│  ├─ questionnaire-fe/          # PC 管理端 + 可视化编辑器 + AI Copilot
│  ├─ questionnaire-client/      # 移动端答题页（Next.js）
│  └─ questionnaire-be/          # NestJS 后端
├─ package.json                  # workspace 根脚本
├─ pnpm-workspace.yaml           # pnpm workspace 声明
└─ lerna.json                    # Lerna 版本管理配置
```

---

## 3. 包职责地图

### 3.1 `packages/questionnaire-fe`

用途：

- PC 管理端
- 问卷编辑器
- AI Copilot 工作台
- 统计页和 AI 分析入口

关键入口：

- 应用入口：`packages/questionnaire-fe/src/main.tsx`
- 路由入口：`packages/questionnaire-fe/src/router/index.tsx`
- 应用壳：`packages/questionnaire-fe/src/App.tsx`

最常看的目录：

- `src/pages/question/Edit/`
  用途：编辑器主页面、AI Copilot、草稿预览
- `src/pages/question/Stat/`
  用途：统计可视化页
- `src/pages/question/AIAnalysis/`
  用途：AI 智能分析页
- `src/store/modules/`
  用途：编辑器状态、页面配置、用户信息等
- `src/apis/modules/`
  用途：前端调用后端接口的封装
- `src/components/QuestionComponents/`
  用途：题型枚举、题型协议、物料三件套基础
- `src/utils/`
  用途：请求、流式解析、题型归一化等通用工具

编辑器和 AI 相关的核心入口：

- 编辑器页面入口：`packages/questionnaire-fe/src/pages/question/Edit/index.tsx`
- AI 工作台主 Hook：`packages/questionnaire-fe/src/pages/question/Edit/hooks/useAiWorkbench.ts`
- AI 协议类型：`packages/questionnaire-fe/src/pages/question/Edit/components/aiCopilotTypes.ts`
- AI 流接口：`packages/questionnaire-fe/src/apis/modules/ai.ts`
- 流式请求工具：`packages/questionnaire-fe/src/utils/streamRequest.ts`
- SSE 帧解析：`packages/questionnaire-fe/src/utils/sseFrameParser.ts`

编辑器状态相关入口：

- 组件状态：`packages/questionnaire-fe/src/store/modules/componentsSlice.ts`
- 页面配置：`packages/questionnaire-fe/src/store/modules/pageConfigSlice.ts`
- 题型协议：`packages/questionnaire-fe/src/components/QuestionComponents/index.ts`

### 3.2 `packages/questionnaire-client`

用途：

- 移动端问卷填写
- 答题状态持久化
- Next.js Route Handlers BFF

关键入口：

- App Router 根布局：`packages/questionnaire-client/app/layout.tsx`
- 首页入口：`packages/questionnaire-client/app/page.tsx`
- 问卷页入口：`packages/questionnaire-client/app/question/page.tsx`
- 问卷页客户端主体：`packages/questionnaire-client/app/question/QuestionnaireClient.tsx`

最常看的目录：

- `app/question/`
  用途：答题页、loading、error、success
- `app/api/questionnaire/route.ts`
  用途：获取问卷数据的 Route Handler
- `app/api/answer/route.ts`
  用途：提交答卷的 Route Handler
- `stores/useAnswerStore.ts`
  用途：Zustand 持久化答题状态
- `components/question-type/`
  用途：移动端各题型渲染组件
- `components/question-ui/`
  用途：题目渲染器、进度条、外层包装

移动端答题状态相关入口：

- 问卷页面：`packages/questionnaire-client/app/question/page.tsx`
- 客户端渲染主体：`packages/questionnaire-client/app/question/QuestionnaireClient.tsx`
- 答案 store：`packages/questionnaire-client/stores/useAnswerStore.ts`
- 题型渲染入口：`packages/questionnaire-client/components/question-type/index.tsx`

### 3.3 `packages/questionnaire-be`

用途：

- 提供问卷、编辑器、答卷、AI、鉴权、文件等后端能力
- 同时挂载 TypeORM、Mongoose、Schedule、静态文件服务

关键入口：

- 服务启动入口：`packages/questionnaire-be/src/main.ts`
- 根模块：`packages/questionnaire-be/src/app.module.ts`

最常看的目录：

- `src/service/editor/`
  用途：编辑器详情获取与保存
- `src/service/answer/`
  用途：答卷收集、统计处理
- `src/service/ai/`
  用途：AI 生成、分析、SSE、会话管理
- `src/service/question/`
  用途：问卷列表和元数据管理
- `src/service/auth/`
  用途：登录、注册、用户
- `src/common/`
  用途：通用 schema、装饰器、工具、响应包装

AI 相关核心入口：

- AI 控制器：`packages/questionnaire-be/src/service/ai/ai.controller.ts`
- AI 主服务：`packages/questionnaire-be/src/service/ai/ai.service.ts`
- 流式 DTO：`packages/questionnaire-be/src/service/ai/dto/copilot-stream.dto.ts`
- 会话实体：`packages/questionnaire-be/src/service/ai/entities/`
- 草稿处理工具：`packages/questionnaire-be/src/service/ai/utils/`

编辑器和统计相关入口：

- 编辑器控制器：`packages/questionnaire-be/src/service/editor/editor.controller.ts`
- 编辑器服务：`packages/questionnaire-be/src/service/editor/editor.service.ts`
- 统计控制器：`packages/questionnaire-be/src/service/answer/answer.controller.ts`
- 统计服务：`packages/questionnaire-be/src/service/answer/answer.service.ts`

---

## 4. 常见功能从哪里开始读

### 4.1 AI Copilot

建议顺序：

1. `packages/questionnaire-fe/src/pages/question/Edit/index.tsx`
2. `packages/questionnaire-fe/src/pages/question/Edit/hooks/useAiWorkbench.ts`
3. `packages/questionnaire-fe/src/apis/modules/ai.ts`
4. `packages/questionnaire-fe/src/utils/streamRequest.ts`
5. `packages/questionnaire-fe/src/utils/sseFrameParser.ts`
6. `packages/questionnaire-be/src/service/ai/ai.controller.ts`
7. `packages/questionnaire-be/src/service/ai/ai.service.ts`
8. `packages/questionnaire-be/src/service/ai/utils/`

### 4.2 编辑器状态管理

建议顺序：

1. `packages/questionnaire-fe/src/pages/question/Edit/index.tsx`
2. `packages/questionnaire-fe/src/store/modules/componentsSlice.ts`
3. `packages/questionnaire-fe/src/store/modules/pageConfigSlice.ts`
4. `packages/questionnaire-fe/src/components/QuestionComponents/index.ts`
5. `packages/questionnaire-fe/src/pages/question/Edit/components/`

### 4.3 移动端答题

建议顺序：

1. `packages/questionnaire-client/app/question/page.tsx`
2. `packages/questionnaire-client/app/question/QuestionnaireClient.tsx`
3. `packages/questionnaire-client/stores/useAnswerStore.ts`
4. `packages/questionnaire-client/components/question-type/`
5. `packages/questionnaire-client/app/api/questionnaire/route.ts`
6. `packages/questionnaire-client/app/api/answer/route.ts`

### 4.4 统计分析

建议顺序：

1. `packages/questionnaire-fe/src/pages/question/Stat/index.tsx`
2. `packages/questionnaire-fe/src/pages/question/AIAnalysis/index.tsx`
3. `packages/questionnaire-be/src/service/answer/answer.controller.ts`
4. `packages/questionnaire-be/src/service/answer/answer.service.ts`
5. `packages/questionnaire-be/src/service/ai/ai.service.ts`

### 4.5 工程化与体验优化

建议顺序：

1. `package.json`
2. `pnpm-workspace.yaml`
3. `lerna.json`
4. `packages/questionnaire-fe/vite.config.ts`
5. `packages/questionnaire-fe/src/router/index.tsx`
6. `packages/questionnaire-fe/src/pages/question/Edit/components/AiMessageList.tsx`
7. `packages/questionnaire-client/stores/useAnswerStore.ts`

---

## 5. 共享概念和跨端连接点

### 5.1 问卷组件协议

最先看：

- 管理端题型协议：`packages/questionnaire-fe/src/components/QuestionComponents/index.ts`
- 管理端组件归一化：`packages/questionnaire-fe/src/utils/normalizeQuestionComponent.ts`
- 后端问卷 detail schema：`packages/questionnaire-be/src/common/schemas/question-detail.schema.ts`
- 移动端题型渲染：`packages/questionnaire-client/components/question-type/`

### 5.2 编辑器和答卷的连接

最先看：

- 编辑器保存：`packages/questionnaire-be/src/service/editor/`
- 移动端获取问卷：`packages/questionnaire-client/app/api/questionnaire/route.ts`
- 答卷提交和统计：`packages/questionnaire-be/src/service/answer/`

### 5.3 AI 和编辑器的连接

最先看：

- 前端 AI 协议类型：`packages/questionnaire-fe/src/pages/question/Edit/components/aiCopilotTypes.ts`
- 前端草稿合并：`packages/questionnaire-fe/src/pages/question/Edit/hooks/aiGenerateDraftMerge.ts`
- 后端草稿规范化：`packages/questionnaire-be/src/service/ai/utils/draft-normalizer.ts`
- 后端差异摘要：`packages/questionnaire-be/src/service/ai/utils/build-diff-summary.ts`

---

## 6. 给后续回顾线程的使用规则

- `ARCHITECTURE.md` 只负责“先去哪读”，不负责替你下现状结论
- 在 `devDoc` 任务看板里登记功能时，后续建议补两个字段：
  - `入口文件`
  - `核心文件`
- `入口文件` 写第一个应该打开的页面 / 路由 / controller / hook
- `核心文件` 写 3-8 个最关键文件，按职责分组，不要一股脑列整个目录

---

## 7. 维护建议

- 新增大模块、包结构调整、路由重构后，同步更新本文件
- 如果某个功能长期找入口困难，优先补这里，而不是把说明塞进 `AGENTS.md`
- `devDoc` 继续承担任务看板、四文档、简历沉淀；本文件不要写成回顾结论库
