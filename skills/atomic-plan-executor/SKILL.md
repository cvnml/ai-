---
name: atomic-plan-executor
description: Legacy 旧入口；不要再把它作为大而全计划执行器使用。用户调用 atomic-plan-executor 时，先判断当前处于哪一段流程，并转向新链路：规划/细化用 plan-mode-plus；已批准 plan 本地化用 modular-plan-docs；成熟阶段执行用 plan-implementation-runner；写文件节点施工图确认用 edit-node-blueprint；阶段状态汇报用 execution-checkpoint；实现链路审查用 implementation-review；前端测试用 frontend-test；日志沉淀 Skill 用 skill-mining-review。本 Skill 不生成计划、不编码、不创建项目本地计划、不做 checkpoint、不做 doc reconcile。
user-invocable: true
---

# Atomic Plan Executor Legacy Entry

## 目标

`atomic-plan-executor` 现在只是旧入口兼容说明，不再作为大而全计划落地执行器使用。

旧版本曾经同时负责：

- 生成计划草稿。
- 保存已批准计划快照。
- 编码前函数级施工图确认。
- 实现执行。
- checkpoint。
- Doc Reconcile。
- diff-to-doc cross-check。
- implementation-to-plan audit。

这些职责现在已经拆分到更小的 Skills 中。继续使用旧的大而全入口，会让计划生成、项目本地文档、执行、审查、测试和文档对账边界混在一起。

## 一句话边界

本 Skill 只做旧入口路由，不生成计划，不编码，不创建项目本地计划，不做 checkpoint，不做 doc reconcile。

## 新链路总览

推荐链路：

```text
plan-mode-plus
→ modular-plan-docs
→ plan-implementation-runner
→ edit-node-blueprint
→ execution-checkpoint
→ implementation-review / frontend-test
```

含义：

1. `plan-mode-plus`：批准前，澄清、调研、查询官方文档、按阶段细化上一版 plan，并判断当前阶段是否可编码。
2. `modular-plan-docs`：批准后，把已批准 plan 落成项目本地模块化计划快照、模块指针、索引和归档入口。
3. `plan-implementation-runner`：按已成熟且已批准的阶段计划连续实现。
4. `edit-node-blueprint`：每个独立写文件节点前，用问题工具展示施工图并让用户校正偏差。
5. `execution-checkpoint`：阶段完成、阻塞、偏离计划或验证降级时输出轻量 checkpoint。
6. `implementation-review`：实现完成后审查真实用户链路、状态同步、异步生命周期和遗漏风险。
7. `frontend-test`：需要前端测试设计、补充和执行时使用。
8. `skill-mining-review`：需要从日志沉淀可复用 Skill 时使用。

## 路由规则

当用户调用或提到 `atomic-plan-executor` 时，先判断用户真正想做哪一步。

### 1. 用户还在规划或细化 plan

转向：`plan-mode-plus`

触发表达：

- “先做计划”。
- “继续细化这个 plan”。
- “review 上一版 plan 能不能直接开写”。
- “把第一阶段细化到可以一次性编码”。
- “这个 plan 还太粗”。

说明：

`plan-mode-plus` 负责读取上一版 plan，保留总阶段地图，按当前阶段做成熟度 review，并用问题工具展示模型判断。

### 2. 用户已经批准 plan，想落成本地模块文档

转向：`modular-plan-docs`

触发表达：

- “这个 plan 已经批准了，帮我落到项目里”。
- “创建项目本地计划快照”。
- “给模块 CLAUDE.md 加计划指针”。
- “归档这个计划”。
- “让下次 /clear 后能恢复这个计划”。

说明：

`modular-plan-docs` 只处理已批准 plan 的项目本地持久化，不生成原生 plan，不编码，不做 doc reconcile。

### 3. 用户要按成熟阶段计划开始实现

转向：`plan-implementation-runner`

触发表达：

- “这个阶段计划已批准，开始实现”。
- “按这个成熟 plan 开始写代码”。
- “继续执行计划里的下一阶段”。
- “不要拆太碎，按阶段推进”。

说明：

`plan-implementation-runner` 只执行已成熟且已批准的阶段计划。如果计划还太粗，应回到 `plan-mode-plus`，不要现场补计划。

### 4. 用户要在某次写文件前确认施工图

转向：`edit-node-blueprint`

触发表达：

- “这次 Edit 前先把要改的函数讲清楚”。
- “准备改 A 文件前，把施工图摊开”。
- “不要把多个 Edit 合并成一个问题”。

说明：

`edit-node-blueprint` 一次问题工具只对应一个写文件节点。多个独立 Edit 必须多次提问；同一次问题工具里的 tabs/问题只拆当前节点，并按函数分工动态生成。

### 5. 用户要看当前执行状态或下一步

转向：`execution-checkpoint`

触发表达：

- “现在做到哪了”。
- “下一步是什么”。
- “这里先暂停，留个 checkpoint”。
- “测试没跑完但要继续”。
- “改动超出计划范围了”。

说明：

`execution-checkpoint` 只输出轻量执行状态，不改代码，不生成新计划，不写长期文档对账。

### 6. 用户要检查实现是否完整

转向：`implementation-review`

触发表达：

- “实现完了，检查有没有漏链路”。
- “这样真的会生效吗”。
- “有没有风险点”。
- “是不是还差一步”。

说明：

`implementation-review` 按真实用户路径审查入口、数据转换、接口请求、状态同步、UI 可见结果、父子组件同步、清理恢复等链路。

### 7. 用户要设计、补充或执行前端测试

转向：`frontend-test`

触发表达：

- “前端页面怎么测”。
- “补一下这个组件测试”。
- “写 Playwright E2E”。
- “这个 Vue2/Element UI 功能要怎么测”。

说明：

`frontend-test` 负责测试设计和执行，尤其适配 Vue2、Webpack、Element UI 老项目。

### 8. 用户要从日志沉淀 Skill

转向：`skill-mining-review`

触发表达：

- “从 edit 日志里提取可复用流程”。
- “把这类问题沉淀成 Skill”。
- “不要写成单次问题总结”。

说明：

`skill-mining-review` 负责从 hook/edit 日志和 transcript 中提取可复用问题解决流程。

## 如果用户坚持使用 atomic-plan-executor

先解释：

```text
atomic-plan-executor 现在是旧入口，不再直接执行完整流程。我会先判断你现在处于哪一步，然后调用对应的新 Skill。
```

然后按路由规则选择新 Skill。

不要继续执行旧的大而全流程。

## 禁止行为

禁止在本 Skill 中：

- 生成原生 plan。
- 细化上一版 plan。
- 创建项目本地计划快照。
- 修改项目源码。
- 调用 Edit / Write / NotebookEdit。
- 做写文件节点施工图确认。
- 输出 checkpoint。
- 做 implementation review。
- 设计或执行前端测试。
- 做 doc reconcile。
- 做 diff-to-doc cross-check。
- 做 implementation-to-plan audit。
- 从日志沉淀 Skill。

## 验证方式

### 结构验证

检查 `SKILL.md` 是否满足：

- frontmatter 有 `name: atomic-plan-executor`。
- description 明确这是 Legacy 旧入口。
- 正文明确新链路。
- 正文明确每个场景应转向哪个 Skill。
- 正文明确本 Skill 不生成计划、不编码、不创建本地计划、不做 checkpoint、不做 doc reconcile。

### 行为 dry run

1. “用 atomic-plan-executor 继续细化这个计划。”
   - 应转向 `plan-mode-plus`。

2. “用 atomic-plan-executor 把批准计划落到项目里。”
   - 应转向 `modular-plan-docs`。

3. “用 atomic-plan-executor 开始实现这个阶段。”
   - 应转向 `plan-implementation-runner`。

4. “用 atomic-plan-executor 在改 A 文件前讲清楚函数。”
   - 应转向 `edit-node-blueprint`。

5. “用 atomic-plan-executor 看现在进度。”
   - 应转向 `execution-checkpoint`。

6. “用 atomic-plan-executor 检查实现有没有漏。”
   - 应转向 `implementation-review`。

7. “用 atomic-plan-executor 设计前端测试。”
   - 应转向 `frontend-test`。
