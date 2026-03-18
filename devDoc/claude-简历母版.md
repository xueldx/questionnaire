# 问卷小筑 · 简历母版（Claude 版）

> 本文件是简历沉淀入口，涵盖所有技术亮点，按模块组织。
> 根据目标岗位从中挑选 3-5 个最相关的点组合。
> 总入口：[[问卷项目_前端暑期实习简历功能梳理]]
> 每个功能默认按“四文档工作流”完成回顾后，再把 `功能复盘` 中的简历表述同步到本文件；必要时也可以吸收 `方案评审` 里的成熟优化表达。

---

## 项目一行介绍

> 基于 React + Next.js + NestJS 的全流程问卷平台，核心亮点是编辑器内嵌 AI Copilot，支持 Prompt 润色→流式生成/修改问卷→内联标注预览→一键应用，并串联移动端填写与 AI 智能分析。

---

## 核心主线

**一份问卷从「AI 润色提示词→流式生成 / 对话式修改 → 内联标注预览 → 可视化编辑 → 发布填写 → AI 智能分析」的全链路前端技术深度。**

面试时不要逐条罗列功能，而是讲一条故事线：用户输入原始需求 → AI 润色提示词（两阶段状态机）→ SSE 流式生成问卷草稿（自定义 block 协议实时解析）→ 内联标注预览对比新旧差异 → 一键应用到编辑器画布 → 继续对话式修改 → 拖拽排序微调 → 发布给用户在移动端填写 → 最后用 AI 分析结果并导出报告。每个环节讲一个你解决的技术难题。

---

## 模块一：AI 问卷助手与智能分析

> AI 助手是本项目简历权重最高的模块，可拆为以下 4 个子亮点独立展开。

### 1A. SSE 流式通信与结构化实时解析

#### 技术方案

- 基于 `fetch` + `ReadableStream.getReader()` 封装通用 `streamRequest` 工具函数（`src/utils/streamRequest.ts`），替代浏览器原生 `EventSource`（原生不支持 POST 和自定义 Header），支持 POST 请求体、JWT 鉴权和 `AbortController` 取消
- 自研 SSE 帧解析器 `sseFrameParser`（`src/utils/sseFrameParser.ts`），实现 `consumeSseBuffer` 缓冲区管理：按 `\n\n` 分割帧边界，逐帧提取 `event:` 和 `data:` 字段，对 `data` 做 JSON 安全解析（失败降级为原始字符串）
- 后端设计自定义 block 协议（`parse-copilot-blocks.ts`）：LLM 输出以 `<<<COMPONENT>>>...<<<END_COMPONENT>>>` 等标记包裹结构化 JSON，后端逐块解析并通过 `draft_partial` 事件增量推送，前端实现渐进式草稿渲染
- 定义 12 种 SSE 事件类型的 TypeScript 联合类型（`AiCopilotStreamEvent`），覆盖 meta / phase / prompt_delta / assistant_delta / tool_call / tool_result / draft_partial / draft / warning / done / error，前端通过事件分发驱动 UI 状态机
- 使用 `requestAnimationFrame` + `bufferedUiUpdatesRef` 缓冲高频流式更新（`useAiWorkbench.ts:584`），将 prompt_delta 和 draft_partial 事件合并到下一帧批量刷新，避免每个 SSE 帧触发一次 React 重渲染

#### 简历精简写法

> 基于 fetch + ReadableStream 封装 SSE 流式通信层，自研帧解析器处理缓冲区分帧与 JSON 安全解析；后端设计自定义 block 协议实现 LLM 输出的结构化增量推送，前端通过 12 种事件类型驱动 UI 状态机，配合 requestAnimationFrame 缓冲批量刷新控制高频重渲染。

#### STAR 表达

- **S**：AI 助手需要实时展示 LLM 的流式输出，包括润色文本、对话回复和结构化问卷草稿，原生 EventSource 不支持 POST 和鉴权
- **T**：设计一套支持鉴权、可取消、能解析结构化数据的 SSE 通信方案，同时控制高频更新带来的渲染性能问题
- **A**：fetch + ReadableStream 替代 EventSource → 自研帧解析器管理缓冲区 → 后端 block 协议增量推送 → 12 种事件类型联合类型 → RAF 缓冲批量刷新
- **R**：流式草稿渲染延迟控制在单帧内，用户可实时看到 AI 逐题生成的问卷内容，无卡顿感

### 1B. 两阶段 Prompt 润色与状态机驱动的生成流程

#### 技术方案

- 设计 generate 模式下的两阶段流程：用户输入原始需求 → AI 润色提示词（polish 阶段）→ 用户确认/编辑润色结果 → AI 正式生成问卷（generate 阶段），润色结果支持用户二次编辑后再提交
- 使用 `useReducer` 实现 `aiGenerateFlowMachine`（`aiGenerateFlowMachine.ts`），定义 10 种 phase（idle / polishing / awaiting_confirmation / connecting / thinking / answering / drafting / completed / cancelled / error）和 9 种 action，确保状态转换可预测
- 实现 Process 步骤可视化（`aiProcessHelpers.ts`）：将 AI 处理过程拆解为 connecting → snapshot → stats → constraints → polishing → thinking → answering → drafting 共 8 个步骤，每个步骤有 pending / running / done / error 四种状态，实时展示在聊天界面中
- 根据 intent（generate / edit）和 stage（polish / generate）动态切换步骤标签和进度文案，同一套 Process 组件适配三种场景（润色 / 生成 / 修改）

#### 简历精简写法

> 设计 Prompt 润色→确认→生成的两阶段流程，基于 useReducer 实现 10 phase × 9 action 的生成流状态机，配合 Process 步骤可视化将 AI 处理过程拆解为 8 个实时追踪步骤，同一套状态机适配润色、生成、修改三种场景。

#### STAR 表达

- **S**：用户直接输入的需求往往不够精确，直接交给 LLM 生成效果差；同时 AI 处理过程对用户是黑盒，等待体验差
- **T**：设计一个让用户参与 Prompt 优化的交互流程，并让 AI 处理过程透明可追踪
- **A**：两阶段流程（polish → confirm → generate）→ useReducer 状态机管理 phase 转换 → Process 步骤可视化（8 步实时追踪）→ 三场景复用
- **R**：润色后的 Prompt 生成质量显著提升，用户可实时看到 AI 在"读取问卷→分析需求→生成草稿"的每一步进展

### 1C. 内联标注预览与草稿合并策略

#### 技术方案

- 实现内联标注预览组件 `AiInlineQuestionnairePreview`（620 行），在原问卷基础上以 Diff 方式标注 AI 建议：5 种标注色调（current 白 / suggestion 绿 / danger 红 / info 蓝 / anchor 橙）分别表示当前题目、AI 新增、AI 删除、AI 信息提示、插入锚点
- edit 模式下：逐题对比 `fe_id` 匹配，检测 type / title / props 变化标记为"AI 建议改为"，未匹配的 draft 组件标记为"AI 建议新增"，未出现在 draft 中的原组件标记为"AI 建议删除"
- generate 模式下：基于当前选中组件计算插入位置（`getGenerateInsertionIndex`），新增题目插入到选中组件之后，未选中时追加到末尾；通过 `buildAddedInsertMap` 构建锚点→新增组件的映射关系
- 前端草稿合并（`aiGenerateDraftMerge.ts`）：`dedupeNewComponents` 去重 + `getGenerateInsertionIndex` 计算插入点 + 三段拼接（前半 + 新增 + 后半）
- 后端草稿规范化（`draft-normalizer.ts`）：对 LLM 输出的组件做类型校验、选项列表归一化（支持 string / object 混合格式）、edit 模式下与 snapshot 做双指针合并保留未修改组件
- 变更摘要生成（`build-diff-summary.ts`）：对比 snapshot 和 draft 输出 added / updated / deleted 三类标签，展示在预览面板顶部

#### 简历精简写法

> 实现内联标注预览组件，以 5 种色调 Diff 标注 AI 对原问卷的新增/修改/删除建议；设计前后端协同的草稿合并策略，前端基于选中组件计算插入位置并去重拼接，后端对 LLM 输出做类型校验和双指针合并，自动生成变更摘要。

#### STAR 表达

- **S**：AI 生成的草稿需要让用户清晰看到"改了什么、加了什么、删了什么"，直接替换会让用户失去对问卷的控制感
- **T**：设计一套内联预览机制，让用户在应用前就能看到 AI 建议与原问卷的逐题对比
- **A**：5 种色调标注系统 → fe_id 匹配检测变更 → 选中组件锚点插入 → 前端去重拼接 + 后端双指针合并 → 变更摘要自动生成
- **R**：用户可以在预览中逐题审查 AI 建议，确认后一键应用到编辑器，保持对问卷内容的完全控制

### 1D. 多轮对话会话管理与 AI 智能分析

#### 技术方案

- 设计完整的会话管理体系：每份问卷可创建多个 AI 会话（conversation），支持 generate / edit 两种 intent，会话持久化到后端数据库（`ai-conversation.entity.ts` + `ai-message.entity.ts`），切换会话时恢复历史消息和草稿状态
- 历史消息规范化（`normalizeConversationMessages`）：将后端返回的 tool_call / tool_result 消息聚合为 Process 步骤卡片，用户看到的是"读取问卷→分析需求→生成草稿"的流程摘要而非原始 tool 调用
- 支持多模型切换：前端从 `/api/ai/models` 动态获取可用模型列表，用户可在生成和分析时选择不同模型
- AI 智能分析页面（`AIAnalysis/index.tsx`）：基于 SSE 流式接收分析结果，输出结构化报告（title / overview / key_insights / question_analyses / recommendations），支持一键导出 Word 报告（HTML → Blob → download）
- 草稿生命周期管理：pending draft（未应用）→ applied（已应用到编辑器）→ discard（放弃），通过 `PendingDraftDecisionModal` 在用户切换模式或发起新请求时提示处理未应用草稿

#### 简历精简写法

> 设计多轮对话会话管理体系，支持会话持久化、历史消息聚合为流程摘要卡片和多模型动态切换；AI 分析端基于 SSE 流式输出结构化报告并支持 Word 导出，草稿生命周期管理确保用户不会意外丢失未应用的 AI 建议。

#### STAR 表达

- **S**：用户可能需要多次与 AI 对话来完善问卷，切换会话后需要恢复上下文；分析结果需要可导出分享
- **T**：设计会话持久化和草稿生命周期管理，让多轮对话体验连贯；提供可导出的结构化分析报告
- **A**：会话 CRUD + 历史消息聚合 → 多模型动态切换 → SSE 流式分析 + 结构化报告 → Word 导出 → 草稿生命周期（pending / applied / discard）
- **R**：用户可跨会话连续优化问卷，切换回历史会话时上下文完整恢复；分析报告可一键导出为 Word 文档

## 模块二：可视化编辑器内核与复杂状态管理

### 技术方案

- 基于统一题型协议抽象 12 类问卷组件，每种组件由渲染器（Component）+ 类型定义（PropsType）+ 配置面板（Config）三件套组成，通过 `ComponentType` 枚举 + `ComponentPropsType` 联合类型实现类型安全的组件注册
- 使用 Redux Toolkit 管理编辑器核心状态，拆分为 `componentsSlice`（组件列表 + 撤销重做）、`pageConfigSlice`（页面配置）等独立 slice，降低复杂交互间的耦合
- 自研 undo/redo 历史栈，维护 `history` 数组 + `historyIndex` 指针，每次操作深拷贝当前组件列表快照入栈；支持非线性撤销——在历史中间位置执行新操作时自动截断后续历史记录
- 实现拖拽排序双引擎：集成 react-beautiful-dnd 完整事件链（DragStart → DragUpdate → DragEnd），同时保留原生 HTML5 Drag API 作为降级方案；针对 React 18 StrictMode 导致的 DOM 同步问题，设计 `enableDrag` 状态 + `setTimeout(50ms)` 延迟重启机制
- 编辑保存携带 `version` 字段，前后端通过乐观锁机制避免旧数据覆盖新数据

### 简历精简写法

> 负责问卷可视化编辑器核心能力建设，基于统一题型 schema 设计可扩展组件体系（渲染器 + 配置面板 + 类型定义三件套），结合 Redux Toolkit 管理编辑器复杂状态，自研 undo/redo 历史栈支持非线性撤销，实现拖拽排序双引擎并处理 React 18 StrictMode 兼容问题，编辑保存通过乐观锁防止并发覆盖。

### STAR 表达

- **S**：问卷编辑页是多交互并存的复杂编辑场景，组件增删改、属性配置、拖拽排序、撤销重做同时存在，状态高度耦合
- **T**：搭建可扩展的编辑器内核，支撑多题型统一渲染与配置，保证状态一致性和操作可回退
- **A**：统一题型协议三件套 → Redux Toolkit 状态分层 → 自研非线性 undo/redo → 拖拽双引擎 + React 18 兼容 → 乐观锁版本控制
- **R**：形成可持续扩展的编辑器内核，新增题型只需实现三个文件，AI 生成、统计分析、移动端渲染都能接入同一套组件协议

## 模块三：移动端答题体验（Next.js App Router）

### 技术方案

- 基于 Zustand + persist 中间件实现答案自动持久化到 localStorage，用户刷新或意外关闭后答题进度不丢失；通过 `partialize` 仅持久化答案数据避免冗余存储
- 设计以问卷 ID 为 key 的多问卷答案 Map（`answersByQuestionnaire`），支持用户在多份问卷间切换互不干扰
- 针对矩阵题的部分填写状态，通过 JSON 中嵌入 `__incomplete__` 标记区分"已保存但未完成"与"已完成"，只有全部行列填写完毕后才计入已完成
- 提交时通过 `getAnsweredStatus` 批量校验所有题目，未答题目自动滚动定位（`scrollIntoView` + CSS 高亮动画）+ 粘性进度条闪烁提示
- 利用 Next.js App Router Route Handlers 作为 BFF 层，服务端直接读写 MongoDB（连接池复用），完成后端数据结构 → 前端组件 Props 的适配转换，前端无需感知存储层

### 简历精简写法

> 移动端基于 Next.js + Zustand 实现答题状态持久化与多问卷隔离，设计矩阵题不完整标记机制精确追踪复杂题型填写状态，提交校验联动滚动定位与高亮动画。通过 Route Handlers 作为 BFF 层完成数据适配，前端无需感知存储层结构。

### STAR 表达

- **S**：移动端答题场景下用户可能中途退出、刷新页面、同时打开多份问卷，矩阵题等复杂题型存在"部分填写"的中间状态
- **T**：设计答题状态管理方案，保证进度不丢失、多问卷互不干扰、提交校验准确
- **A**：Zustand + persist 持久化 → 问卷 ID 为 key 的多问卷 Map → `__incomplete__` 标记矩阵题中间状态 → 批量校验 + 滚动定位 + 高亮动画 → Route Handlers BFF 层数据适配
- **R**：用户刷新页面后答题进度完整保留，复杂题型填写状态准确追踪，提交校验零遗漏

---

## 模块四：多题型统计可视化

### 技术方案

- 基于题型分发机制，为不同题型映射差异化统计组件：单选/多选 → 饼图/柱状图，评分/NPS → 分布图，矩阵题 → 热力图，文本题 → 词云，日期题 → 时间分布
- 服务端按题目维度聚合答卷结果，对不同题型采用不同统计逻辑（选项计数、均值计算、矩阵交叉统计等），返回增强后的统计数据
- 前端图表层补充降级方案，词云渲染失败时自动回退到柱状图
- 统计页与 AI 分析串联，图表可视化和智能总结形成数据洞察闭环

### 简历精简写法

> 围绕问卷题型设计差异化统计可视化方案，按题目类型分别渲染饼图、柱状图、矩阵热力图、词云等视图，加入降级容错机制，并与 AI 分析能力联动形成数据洞察闭环。

---

## 模块五：工程化与体验优化

### 技术方案

- pnpm Workspace + Lerna 管理 Monorepo，Vite + SWC 构建 PC 端，Next.js 构建移动端
- PC 端基于 Redux Toolkit（编辑器需要 undo/redo），移动端基于 Zustand + persist（答卷需要持久化和轻量）——场景化状态管理选型
- Vite `manualChunks` 分包策略，将大依赖拆为独立 chunk 利用浏览器缓存
- 编辑器场景 `React.memo` 精细化控制重渲染，AI 流式场景节流控制更新频率
- `[待补]` 路由级 `React.lazy` + `Suspense` 懒加载、虚拟列表、Lighthouse 量化指标前后对比

### 简历精简写法

> 基于 pnpm Monorepo 管理多应用工程，根据场景选型状态管理方案（编辑器用 Redux Toolkit，答卷用 Zustand），通过 Vite 分包、React.memo 精细化渲染控制和流式更新节流提升系统性能与可维护性。

---

## 可直接裁剪的简历版本

### 版本 A：AI 交互体验型（适合 AI 应用 / LLM 产品方向）

- 设计编辑器内嵌 AI Copilot，支持 Prompt 润色→流式生成/对话式修改问卷→内联标注预览→一键应用的完整交互链路；基于 useReducer 实现两阶段生成流状态机（10 phase × 9 action），配合 Process 步骤可视化让 AI 处理过程透明可追踪
- 基于 fetch + ReadableStream 封装 SSE 流式通信层，后端设计自定义 block 协议（<<<COMPONENT>>>标记）实现 LLM 输出的结构化增量推送，前端通过 12 种事件类型驱动 UI 状态机，requestAnimationFrame 缓冲批量刷新控制高频重渲染
- 实现内联标注预览组件，以 5 种色调 Diff 标注 AI 对原问卷的新增/修改/删除建议，前后端协同的草稿合并策略支持去重拼接和双指针合并，自动生成变更摘要
- 设计多轮对话会话管理体系，支持会话持久化、多模型动态切换、AI 智能分析流式输出结构化报告与 Word 导出

### 版本 B：复杂前端编辑器型（适合低代码 / 中后台 / 编辑器方向）

- 负责问卷可视化编辑器核心能力建设，基于统一题型 schema 抽象 12 类组件（渲染器 + 配置面板 + 类型定义三件套），结合 Redux Toolkit 管理编辑器复杂状态，支撑组件增删改、画布渲染、移动端展示与统计分析链路
- 自研 undo/redo 历史栈支持非线性撤销，实现拖拽排序双引擎（react-beautiful-dnd + 原生 HTML5 降级）并处理 React 18 StrictMode 兼容问题，编辑保存通过 version 字段实现乐观锁防止并发覆盖
- 在编辑器中嵌入 AI Copilot，通过 SSE 流式通信 + 自定义 block 协议将 LLM 输出结构化回填编辑器画布，内联标注预览以 Diff 方式展示 AI 建议，草稿合并策略支持去重和位置感知插入
- 打通 PC 管理端创建编辑、移动端填写和答卷统计分析的全流程链路，围绕统一题型协议完成多端渲染与答案回收

### 版本 C：全链路工程化型（适合大厂通用前端 / 全栈方向）

- 基于 React + Next.js + NestJS 搭建全流程问卷平台，采用 pnpm Monorepo 管理三端应用，根据场景选型状态管理方案（编辑器用 Redux Toolkit 支撑 undo/redo，答卷用 Zustand + persist 实现持久化）
- PC 端负责可视化编辑器核心能力（组件 schema 设计、拖拽排序、撤销重做、乐观锁版本控制），编辑器内嵌 AI Copilot 支持流式生成/修改问卷与内联标注预览
- 移动端基于 Next.js 实现答题状态持久化与多问卷隔离，通过 Route Handlers 作为 BFF 层完成数据适配
- 围绕问卷题型设计差异化统计可视化方案，按题目类型映射饼图、柱状图、矩阵热力图、词云等视图，与 AI 分析联动形成数据洞察闭环

---

## 面试最强点回答模板

> 这个项目我最想强调的是编辑器内嵌 AI Copilot 的完整设计。从 SSE 流式通信层（自研帧解析器 + 自定义 block 协议实现结构化增量推送），到两阶段 Prompt 润色状态机（useReducer 驱动 10 个 phase 转换），再到内联标注预览（5 种色调 Diff 标注 + 前后端协同草稿合并），这条链路把 AI 能力真正嵌入了编辑器的交互流程里，而不只是"调个接口展示结果"。底层支撑这一切的是统一题型 schema 和可扩展的组件协议——AI 生成、编辑器渲染、移动端展示、统计分析都接在同一套数据结构上。
