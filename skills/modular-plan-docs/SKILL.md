---
name: modular-plan-docs
description: 在原生 plan mode 计划已被用户批准后使用；把 Claude 侧已批准 plan 转换为项目本地模块化计划文档体系。负责判断 owning module、创建或更新模块共址计划快照、维护模块 CLAUDE.md 指针、根级索引、归档状态和跨上下文恢复入口；不生成原生 plan、不同步未批准草稿、不创建 planning-with-files 三件套、不进入实现、不写函数级施工图、不做 diff-to-doc 审计。
user-invocable: true
---

# Modular Plan Docs

## 目标

`modular-plan-docs` 负责把已经批准的 Claude 侧 plan 落成项目本地、模块共址、可索引、可归档、可跨上下文恢复的计划文档体系。

它解决的问题是：原生 plan 文件适合当轮审批和讨论，但复杂任务在换会话、上下文压缩、多人协作或长期暂停后，容易丢失边界、阶段、验收标准和下一步。项目本地计划快照用于提供长期恢复入口。

重点原则：

- 原生 plan 文件是审批和当轮讨论入口。
- 项目本地计划快照是跨上下文恢复入口。
- 模块计划应优先与 owning module 共址。
- 根级文档只做索引和地图，不承载完整模块计划。

## 一句话边界

只做“已批准 plan 的模块化本地文档落地”，不做 plan 生成，不做实现，不做 doc reconcile。

## 什么时候使用

当用户表达以下意图时使用：

- “这个 plan 已经批准了，帮我落到项目里”。
- “把已批准 plan 转成模块计划文档”。
- “为这个计划创建项目本地计划快照”。
- “给模块 CLAUDE.md 加计划指针”。
- “整理项目本地计划索引”。
- “归档这个计划”。
- “让下次 /clear 或新会话后能恢复这个计划”。

## 什么时候不要使用

不要在这些场景使用本 Skill：

- 用户还没有批准原生 plan。
- 用户只是要求“先规划一下”。
- 用户要求生成原生 plan。
- 用户要求编码或执行计划。
- 用户要求创建 `task_plan.md`、`findings.md`、`progress.md` 三件套。
- 用户要求根据实现 diff 做文档审计。
- 用户要求实现完成后的 doc reconcile。
- 用户只是要临时总结当前对话，不需要项目本地恢复入口。

如果用户还没有批准 plan，应停止并提示用户先完成 `plan-mode-plus` 或原生 plan mode 的批准流程。

## 输入前置条件

必须满足至少一个条件：

- 用户提供已批准 plan 文件路径。
- 当前对话中有明确已批准 plan 内容。
- 用户明确说明某个 Claude 侧 plan 已批准。

如果没有批准证据，不要把想法、草稿或未确认方案写入项目本地文档。

## 工作总流程

固定流程：

```text
确认 plan 已批准
→ 读取已批准 plan
→ 判断 owning module
→ 选择本地文档落点
→ 创建/更新计划快照
→ 更新模块 CLAUDE.md 指针
→ 必要时更新根级索引
→ 标记状态和恢复入口
→ 停止，不进入实现
```

完成计划文档落地后必须停下，不要自动进入编码、测试修复、checkpoint 或 doc reconcile。

## 第 1 步：确认 plan 已批准

先确认当前要落地的是已经批准的 plan，而不是计划草稿。

可接受的批准信号：

- 用户明确说“已批准”“可以按这个计划落地”“这个 plan 通过了”。
- 当前原生 plan mode 已经通过审批并退出。
- 用户提供的 plan 文件路径明确指向已批准计划，并说明要落成本地文档。

不可接受的信号：

- 用户只是说“帮我想想”。
- 用户只是贴了一个未确认方案。
- 当前 plan 仍处在等待确认状态。
- 计划里还存在未解决的关键分叉。

如果批准状态不清楚，要先用普通语言提问确认，不要猜。

## 第 2 步：读取已批准 plan

读取 plan 时只提取适合长期恢复的内容：

- 用户已批准的真实目标。
- 已批准范围和明确不做项。
- 已确认的关键选择。
- 阶段路线和完成标志。
- 验证建议。
- 风险和边界。

不要复制：

- 大段推理过程。
- 临时工具输出。
- 未批准草稿内容。
- 函数级施工细节。
- 测试失败流水账。

## 第 3 步：判断 owning module

owning module 是计划快照的主要归属模块。它决定计划应该共址到哪里，后续新会话也从哪里恢复。

判断顺序：

1. 主要用户可见功能属于哪个模块。
2. 主要代码改动预计落在哪个目录。
3. 哪个模块拥有相关 `CLAUDE.md`、README 或模块文档。
4. 哪个模块定义关键接口、状态或数据流。
5. 如果涉及多个模块，哪个模块是入口或协调方。
6. 如果仍不清楚，向用户确认，不要猜。

多模块计划只选一个 primary owning module。其他模块只写 related modules、必要指针或协作约束，不复制完整计划。

## 第 4 步：选择本地文档落点

计划快照路径优先级：

1. 单模块任务：`<owning-module>/plans/<slug>.md`
2. 如果项目不希望新增 `plans/` 目录：`<owning-module>/PLAN.<slug>.md`
3. 跨模块但有主模块：快照仍放 primary owning module，根级只放索引。
4. 真正项目级计划：才使用项目根 `plans/<slug>.md` 或 `PLAN.<slug>.md`。

不要默认把所有计划放项目根。

slug 规则：

- 用短横线命名。
- 表达业务目标，不写日期流水号作为唯一含义。
- 避免包含客户敏感信息、账号、token、内部地址。

## 第 5 步：创建或更新计划快照

计划快照用于恢复上下文和指导后续执行方向，不是完整施工图。

模板：

```markdown
# Plan Snapshot: <计划标题>

## Status

- State: active
- Source plan: <Claude 侧原生 plan 路径或 current approved conversation plan>
- Approved at: <YYYY-MM-DD>
- Owning module: <模块路径>
- Related modules:
  - <模块路径或“无”>
- Resume entry:
  - Start from: <当前阶段或下一步>
  - Read first: <当前计划快照路径>
  - Then read: <模块 CLAUDE.md 路径>

## 1. Approved Goal

<用简短业务语言说明用户已经批准的目标。>

## 2. Scope

### In scope

- <已批准范围 1>
- <已批准范围 2>

### Out of scope

- 不生成原生 plan
- 不执行实现
- 不写函数级施工图
- 不做 doc reconcile
- 不做 diff-to-doc 审计
- <该计划明确不做的其他事项>

## 3. Confirmed Decisions

- <用户已确认的关键选择>
- <如果没有关键分叉，写“无额外关键分叉；沿用现有模块模式。”>

## 4. Module Ownership

- Primary owning module: `<模块路径>`
- Why this module owns the plan: <一句话说明归属依据>
- Cross-module notes:
  - <如果涉及其他模块，写接口、依赖或协作边界>
  - <不涉及则写“无”>

## 5. Phase Map

### Phase 1: <阶段名>

Goal: <阶段目标>

Planned actions:
- <动作 1>
- <动作 2>

Completion signal:
- <怎样算完成>

## 6. Validation

- <自动化验证建议>
- <手动验证建议>
- <回归检查建议>

## 7. Recovery Notes

If context is lost:

1. Read this file.
2. Read `<模块 CLAUDE.md 路径>`.
3. Continue from `<当前阶段 / 下一步>`.
4. Do not re-open plan generation unless the approved scope has changed.

## 8. Archive Log

- <YYYY-MM-DD>: Created from approved Claude plan.
```

模板原则：

- 轻量，便于快速恢复。
- 面向恢复和执行方向。
- 不复制原生 plan 全文。
- 不记录每次工具调用。
- 不写函数级施工图。
- 不把未确认猜测写成事实。

## 第 6 步：更新模块 CLAUDE.md 指针

模块 `CLAUDE.md` 只放索引，不复制完整计划。

如果模块 `CLAUDE.md` 已存在 `## Plans`，在对应状态分组下更新计划指针。如果不存在，可新增一个轻量 `## Plans` 区块。

模板：

```markdown
## Plans

### Active

- `<plans/<slug>.md>` — <一句话说明目标>；state: active；resume from: <阶段名或下一步>.

### Paused

- `<plans/<slug>.md>` — <暂停原因>；state: paused；resume condition: <恢复条件>.

### Archived

- `<plans/archive/<slug>.md>` — <完成/废弃/被替代原因>；state: completed | superseded | abandoned.
```

模块 `CLAUDE.md` 只写：

- 计划快照路径。
- 当前状态。
- 一句话目标。
- 恢复入口。
- 与模块长期边界相关的最小说明。

不写：

- 完整计划。
- 函数级施工图。
- 每次尝试记录。
- 测试失败流水账。
- 临时环境问题。
- 未批准草稿。

## 第 7 步：必要时更新根级索引

根级 `CLAUDE.md` 只在必要时维护索引。

适用场景：

- 跨模块计划。
- 项目级计划。
- 根级已经有模块文档地图。
- 用户明确要求根级有入口。

模板：

```markdown
## Project Plan Index

Root-level index only. Detailed plans live beside their owning modules.

### Active cross-module plans

- `<module-path>/plans/<slug>.md>` — <一句话说明跨模块目标>；primary owner: `<module-path>`；related: `<other-module>`.

### Module plan indexes

- `<module-a>/CLAUDE.md#plans`
- `<module-b>/CLAUDE.md#plans`
```

根级不写：

- 模块内部完整计划。
- 阶段细节。
- 函数级施工图。
- 过程日志。
- 未批准草稿。

## 第 8 步：归档计划

归档状态枚举：

```text
active
paused
completed
superseded
abandoned
```

默认归档路径：

```text
<module>/plans/archive/<slug>.md
```

归档时更新三处：

1. 计划快照自身：更新 State，追加 Archive Log。
2. 模块 `CLAUDE.md`：从 Active/Paused 移到 Archived。
3. 根级索引：仅当根级已有该计划入口时更新。

归档记录模板：

```markdown
## Archive Log

- <YYYY-MM-DD>: Created from approved Claude plan.
- <YYYY-MM-DD>: Archived as completed. Outcome: <一句话结果>.
```

被替代：

```markdown
- <YYYY-MM-DD>: Archived as superseded. Replaced by `<new-plan-path>`.
```

废弃：

```markdown
- <YYYY-MM-DD>: Archived as abandoned. Reason: <一句话原因>.
```

暂停时不移动文件，只更新状态和恢复条件。

## 与现有 Skills 的关系

### plan-mode-plus

- `plan-mode-plus`：批准前，增强原生 plan mode，生成 Claude 侧可审批 plan。
- `modular-plan-docs`：批准后，把已批准 plan 落成项目本地模块化计划文档。

硬边界：

```text
本 Skill 不创建或修改原生 plan mode 当前计划草稿；它只在用户明确表示计划已批准后，把该计划整理为项目本地模块化计划文档。
```

### atomic-plan-executor

只复用文档层级思想，不复用执行器职责。

不做：

- Implement。
- 编码前函数级施工图。
- checkpoint。
- Doc Reconcile。
- diff-to-doc cross-check。
- implementation-to-plan audit。

### planning-with-files

不创建：

- `task_plan.md`
- `findings.md`
- `progress.md`

本 Skill 优先模块共址，而不是根目录三文件系统。

### skill-mining-review

不读取 edit logs，不生成 skill candidates，不判断是否升级正式 Skill。

如果用户要求“从日志沉淀 Skill”，应使用 `skill-mining-review`，不要用本 Skill。

## 反模式

禁止这些行为：

- 用户还没批准 plan，就同步到项目文档。
- 把原生 plan 草稿当作已批准计划。
- 把未确认分叉写进本地计划快照。
- 在根级 `CLAUDE.md` 写完整模块计划。
- 在多个模块复制同一份完整计划。
- 把计划快照写成函数级实现文档。
- 创建 `task_plan.md`、`findings.md`、`progress.md`。
- 计划落地后自动开始编码。
- 完成计划落地后做 diff-to-doc 审计。
- 把实现过程中的测试失败、报错、命名变化写入长期文档。
- 把业务敏感信息、账号、token 或内部地址写进 slug 或索引摘要。

## 验证方式

### 结构验证

检查 `SKILL.md` 是否满足：

- frontmatter 有 `name: modular-plan-docs`。
- description 明确“已批准 plan 后使用”。
- description 明确负面边界：不生成原生 plan、不同步未批准草稿、不创建 planning-with-files 三件套、不进入实现、不写函数级施工图、不做 diff-to-doc 审计。
- 正文有 owning module 判断规则。
- 正文有计划快照模板。
- 正文有模块 `CLAUDE.md` 指针模板。
- 正文有根级索引规则。
- 正文有归档规则。
- 正文有跨上下文恢复入口规则。

### 行为 dry run

用这些模拟请求检查边界：

1. “帮我把这个想法整理成模块计划文档。”
   - 如果没有已批准 plan，应停止并要求先完成 plan 批准。

2. “这个 plan 已经批准了，帮我落到 apps/electron-shell 的模块计划文档。”
   - 应读取已批准 plan。
   - 应判断 owning module。
   - 应推荐或创建模块共址计划快照。
   - 应更新模块 `CLAUDE.md#Plans` 指针。
   - 不应进入实现。

3. “这个跨 electron-shell 和 ts-gateway 的 plan 已批准，帮我做本地模块化计划。”
   - 应选择 primary owning module。
   - 其他模块只写 related modules 或必要指针。
   - 根级只写路径和摘要。

4. “把这个已批准计划转成 task_plan.md、findings.md、progress.md。”
   - 不应由本 Skill 创建三件套。
   - 应说明这是 planning-with-files 的范围。

5. “计划文档落好后直接开始写代码。”
   - 完成计划文档落地后停止。
   - 明确本 Skill 不进入实现。

6. “这个模块计划已经完成，帮我归档。”
   - 应更新状态、Archive Log、模块指针，必要时更新根级索引。
   - 不做 diff-to-doc 审计。

### 边界回归

确认不会发生：

- 写入原生 plan mode 当前计划文件。
- 把未批准 plan 写入项目。
- 创建 `task_plan.md`、`findings.md`、`progress.md`。
- 修改源码。
- 安装依赖。
- 运行实现命令。
- 写函数级施工图。
- 根据 git diff 审计文档。
- 把完整计划复制到根级 `CLAUDE.md`。
