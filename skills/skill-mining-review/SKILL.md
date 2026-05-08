---
name: skill-mining-review
description: 从 Claude 日常修改日志中提取可复用的问题解决流程，按触发场景、判断信号、操作步骤、验证方式聚类为 skill_topic，并判断应丢弃、写入 CLAUDE.md、合并已有 Skill、创建项目 Skill 或创建全局 Skill。不要把 Skill 写成单次问题总结。
---

# Skill Mining Review

这个 Skill 用来把日常修改记录里反复出现的“解决问题流程”转成可管理的技能候选池，而不是把某一次问题分析写成总结，也不是直接生成大量正式 Skill。

## 什么时候使用

当用户要求以下事情时使用：

- 从 edit 日志里提取可复用的问题解决流程
- 从日常修改里沉淀“遇到什么信号、怎么判断、按什么步骤处理、如何验证”的做事方法
- 判断某些修改是否适合变成 Skill，而不是只总结这次问题
- 把多个候选按触发场景、判断信号、操作步骤或验证方式聚类
- 审查 skill-candidates 目录，决定是否生成或更新正式 Skill
- 用户提到“不要一条候选一个 Skill”“同类候选合并成一个 Skill”“Skill 不是问题总结”

## 输入材料优先级

优先读取当前项目的这些文件：

1. `.claude/logs/skill-mining-pending.json`：如果存在，说明 edit 日志已达到阈值，需要优先处理其中 `processRange` 指定的记录
2. `.claude/logs/edit-tools.index.jsonl`：轻量索引，先用它筛选高价值记录
3. `.claude/logs/edit-tools.jsonl`：完整修改原文，只有需要追溯细节时再读取
4. `transcript_path` 指向的会话记录：需要理解用户问题、失败尝试、最终结论时读取
5. `.claude/skill-candidates/topics/`：已有候选主题池
6. `.claude/skills/` 和 `~/.claude/skills/`：已有正式 Skill，避免重复创建
7. `CLAUDE.md`：项目已沉淀规则，避免重复写候选

## Hook 日志的定位

这个 Skill 分析的主要输入通常是由 Claude Code hooks 自动记录的 edit 日志，例如：

```text
.claude/logs/edit-tools.index.jsonl
.claude/logs/edit-tools.jsonl
```

这些日志的本质是“修改行为证据”，不是现成的 Skill。它们通常只能直接说明：

- 什么时候改了哪个文件
- 使用了哪个工具：`Write`、`Edit` 等
- 修改属于什么粗分类：`feature-implementation`、`technical-issue`、`tooling-config` 等
- 修改价值判断：`high`、`medium`、`low`
- 修改前后的文本片段
- 对应的 `transcript_path`，可追溯当时用户问题和 Claude 的分析过程

因此不能把一条日志直接总结成 Skill。正确做法是：

```text
index 日志筛选高价值记录 → full 日志查看具体改动 → transcript 还原问题上下文和推理过程 → 抽出可复用流程 → 聚合成 skill_topic
```

## 核心原则

候选经验可以很多，但正式 Skill 必须少而精。

Skill 的本质是可复用流程，不是问题复盘。提取时要优先寻找：

```text
触发场景 → 判断信号 → 操作步骤 → 验证方式 → 不适用边界
```

不要执行：

```text
一条候选经验 → 一个 Skill
一次问题现象 → 一篇 Skill
把“这次怎么修好”写成问题总结
```

应该执行：

```text
多条同类候选经验 → 一个 skill_topic → 一个可复用的问题解决流程 → 一个成熟 Skill
```

如果日志只能说明“发生了什么”和“改了什么”，但不能提炼出下次遇到同类问题时该怎么判断、怎么做、怎么验证，就不要升级为正式 Skill。

从 hook 日志提炼时，要把字段映射成流程线索：

| 日志字段 | 能说明什么 | 用途 |
|---|---|---|
| `category` / `value` | 记录是否值得深挖 | 决定优先级，不直接决定能否成 Skill |
| `target_path` / `ext` / `tech` | 问题发生在哪类技术栈和文件 | 帮助归类 skill_topic |
| `old_string` / `new_string` | 具体改法 | 只能作为处理步骤证据，不能直接当流程 |
| `searchableText` / `tags` | hook 粗分类和检索词 | 用来批量聚类相似记录 |
| `transcript_path` | 当时的问题描述、判断过程、验证反馈 | 用来补齐触发场景、判断步骤和验证方式 |

## transcript_path 读取流程

当候选记录可能升级为 Skill，或者仅靠 `old_string` / `new_string` 无法判断可复用流程时，必须读取对应日志里的 `transcript_path`。

读取目标不是复述整段对话，而是提取这些信息：

1. 用户最初描述的真实问题或目标
2. Claude 当时先做了哪些判断和排查
3. 中间有没有失败尝试、用户纠正或方向调整
4. 最终采用的解决路径是什么
5. 是否执行过测试、页面验证、命令验证或用户确认
6. 哪些内容只是当前项目业务细节，不能泛化进 Skill

读取后要把会话内容转成流程字段：

| transcript 中的信息 | 转成候选 Skill 的字段 |
|---|---|
| 用户原始需求、报错、现象 | 适用场景、触发信号 |
| Claude 的排查顺序 | 判断步骤 |
| 最终修改方案 | 处理步骤 |
| 测试结果、页面操作、用户确认 | 验证方式 |
| 业务字段、客户定制、临时代码 | 不应该泛化的业务细节、不适用边界 |

如果 `transcript_path` 不存在、无法读取，或者会话里没有足够上下文，必须在候选里标记为“上下文不足”，不要升级为正式 Skill。

## 分类规则

读取日志索引时，优先关注：

```text
value:high
value:medium
category:technical-issue
category:feature-implementation
category:debugging-process
category:performance
category:tooling-config
```

通常降低优先级：

```text
category:ordinary-log
category:business-change
change:temporary
value:low
```

业务变更不是没有价值，但默认不要直接做成全局 Skill。业务规则更适合：

- 写入项目 CLAUDE.md
- 写入项目 Skill
- 归入项目专属 skill_topic

## skill_topic 聚类方法

给每条候选分配一个稳定的 `skill_topic`。

`skill_topic` 应该描述可复用的处理流程，而不是某次具体 bug。命名时优先表达“哪类场景下的解决方法”，不要表达“某个问题的标题”。

好例子：

```text
vue2-resource-lifecycle
vue2-form-payload-sync
cesium-layer-lifecycle
webpack4-legacy-build-debug
claude-code-hook-debug
java-threadpool-risk
java-concurrency-locking
api-request-debug
```

坏例子：

```text
fix-message-vue-bug-0429
newtask-43-speed-field
button-click-error
some-user-request-fix
```

## 候选卡片结构

生成候选时，写入：

```text
.claude/skill-candidates/topics/<skill_topic>/candidates/<date>-<short-name>.md
```

候选卡片格式：

```markdown
---
skill_topic: vue2-resource-lifecycle
category: technical-issue
value: high
status: pending-review
reuse_scope: project-or-global
source_logs:
  - .claude/logs/edit-tools.index.jsonl#L10
source_files:
  - src/example.vue
tech:
  - vue2
  - event-bus
---

# 候选流程：<一句话描述可复用场景>

## 适用场景

用普通语言描述下次遇到什么类型的问题时应该想起这个流程。

## 触发信号

列出能提示 Claude 应该启动这个流程的现象、报错、用户表述、代码形态或日志特征。

## 判断步骤

说明先看什么、再排除什么、如何确认根因，不要只写这次问题的结论。

## 处理步骤

把解决方式写成可复用操作顺序；需要细节时引用完整日志，不要在候选里复制大段代码。

## 验证方式

说明如何确认问题真的解决，包括测试、页面操作、命令输出、日志观察或回归检查。

## 不适用边界

说明哪些相似问题不应该套用这个流程，避免 Skill 误触发。

## 不应该泛化的业务细节

列出只属于当前项目、当前客户、当前字段、当前业务类型码的内容。

## 建议去向

- 丢弃
- 写入 CLAUDE.md
- 合并已有 Skill
- 创建项目 Skill
- 创建全局 Skill
```

## 判断是否升级为正式 Skill

只有同时满足“流程完整”和下面条件之一，才建议创建或更新正式 Skill。

流程完整指候选里至少说清楚：

1. 什么时候触发
2. 先检查什么信号
3. 按什么顺序处理
4. 怎么验证结果
5. 什么时候不要套用

升级条件：

1. 同一 `skill_topic` 下候选数量不少于 3 条
2. 同一 `skill_topic` 下候选数量不少于 2 条，且至少一条是 `value:high`
3. 用户明确要求把该主题沉淀成 Skill
4. 该主题明显跨项目可复用，且触发条件清楚

不要因为一次偶然修改就创建 Skill；也不要把一次完整的问题分析直接改名成 Skill。

## 决策去向

对每个候选或 topic，给出一个建议：

### 丢弃

适合：

- 临时测试
- 文案微调
- 单纯格式调整
- 没有复用价值的一次性修改

### 写入 CLAUDE.md

适合：

- 当前项目稳定事实
- 项目业务规则
- 项目目录、路由、接口、权限、类型码说明
- 需要所有后续任务都知道的项目背景

### 合并已有 Skill

适合：

- 已经有同类 Skill
- 只是给已有 Skill 增加一个检查点、反例或适用场景

这是优先选择。

### 创建项目 Skill

适合：

- 只适用于当前项目
- 和当前项目架构、业务流程、大文件模式强相关
- 例如无人机任务类型、Cesium 图层、消息通知分组

### 创建全局 Skill

适合：

- 跨项目可复用
- 不依赖当前项目的业务类型码、客户字段、接口路径
- 例如 Vue2 生命周期清理、Webpack 4 排错、Java 并发风险检查

## 生成正式 Skill 前的检查

如果建议生成或更新 Skill，必须先检查：

1. 是否已有同名或同主题 Skill
2. 是否和已有 Skill 描述重叠
3. 是否包含业务敏感信息
4. 是否包含临时测试内容
5. 是否有明确触发条件
6. 是否有可执行的判断步骤、处理步骤和验证方式
7. 是否有明确不适用场景
8. 是否会让 Claude 在其他任务中误触发
9. 是否只是一次问题总结；如果是，继续沉淀为候选，不要升级为 Skill

## 满 100 条日志后的 pending 处理

如果 `.claude/logs/skill-mining-pending.json` 存在，按下面流程处理：

1. 读取 pending 文件中的 `processRange.fromLine` 和 `processRange.toLine`
2. 只处理这个范围内的 `.claude/logs/edit-tools.index.jsonl` 记录
3. 需要细节时，再读取 `.claude/logs/edit-tools.jsonl` 中相同范围的记录
4. 按 `skill_topic` 生成或更新候选卡片
5. 聚合成功后，如果 pending 的 `deleteMode` 是 `direct-delete-after-aggregation`，从 `edit-tools.index.jsonl` 和 `edit-tools.jsonl` 中删除已处理范围内的记录
6. 删除 `.claude/logs/skill-mining-pending.json`
7. 写入 `.claude/logs/skill-mining-processed.jsonl`，只记录处理摘要，不复制完整 `old_string`、`new_string`、`content`

删除日志前必须满足：

- 已经生成候选卡片，或者明确判断该范围内没有值得沉淀的候选
- 输出里说明删除了哪一个 line 范围
- 不删除 pending 范围之外的新日志

## 输出要求

默认不要直接创建正式 Skill，除非用户明确要求。

默认输出：

```markdown
## 技能候选提取结果

### 候选概览

| skill_topic | 候选数 | 最高价值 | 建议去向 | 原因 |
|---|---:|---|---|---|

### 新增候选

- `<skill_topic>`：<候选流程标题> → <建议>
  - 触发场景：...
  - 核心流程：判断 → 处理 → 验证

### 建议升级的 topic

- `<skill_topic>`：建议合并/创建 `<skill-name>`，原因是它已经形成可复用流程：<触发条件> → <判断步骤> → <处理步骤> → <验证方式>

### 不建议沉淀的记录

- <原因>

### 下一步

等待用户确认后，再创建或更新正式 Skill。
```

## 重要约束

- 不要把完整 `old_string`、`new_string`、`content` 大段复制进候选卡片。
- 候选卡片只写摘要和引用，完整原文留在日志里。
- 不要把业务类型码、客户定制字段误写进全局 Skill。
- 优先合并已有 Skill，少建新 Skill。
- Skill 是长期能力，不是日志归档。
- Skill 要沉淀“下次怎么做”，不是复述“这次发生了什么”。
- 如果只能写出问题背景、原因分析、修改结果，说明它还只是候选经验，不是成熟 Skill。
