# 模块执行文档：<模块名>

> legacy / optional reference only。
> 这是旧的复杂模块辅助文档模板，不再是默认入口。
> 默认情况下应直接遵循 `SKILL.md` 的去模板化规则，把详细持久化计划写入 owning module 的 `CLAUDE.md`。

## 模块信息

- 名称：`<模块名>`
- 类型：`<shared / backend / frontend>`
- 默认粒度：`单功能优先`
- 当前状态：`规划中 / 执行中 / 已阻塞 / 已完成`
- 对应 `CLAUDE.md` 小节：`<section-name>`

## 当前快照

- 当前模式：`plan-sync / execute / complex-module-assist`
- 当前任务：`<task-id or title>`
- 上轮完成：`<one-line summary>`
- 下一步：`<one-line next action>`
- 阻塞项：`<none / blockers>`
- 约束同步状态：`synced / needs-update`
- 最近 plan/code 对账：`YYYY-MM-DD HH:mm / none`
- 最近 implementation-to-plan audit：`YYYY-MM-DD HH:mm / none`
- 最近 diff-to-doc cross-check：`passed / pending / needs-update`
- 最后更新：`YYYY-MM-DD HH:mm`

## 模块目标

一句话写清楚：这个模块最终要解决什么问题。

## 当前范围

这一轮明确纳入：
- 范围 1
- 范围 2

这一轮明确不纳入：
- 非目标 1
- 非目标 2

## Business Invariants

> 只记录本复杂模块内的关键业务不变量。
> 跨模块或长期有效规则必须同步到 `CLAUDE.md`。

| ID | 规则 | 适用范围 | 来源 | 实现位置 | 状态 |
|---|---|---|---|---|---|
| BI-1 | <任何情况下必须成立的规则> | <module/function> | <user/plan/code/audit> | <file/function> | <active/synced> |

## Function/Method Constraint Ledger

| 函数/方法/接口 | 约束内容 | 关联规则 | 实现位置 | 验证方式 |
|---|---|---|---|---|
| <name> | <输入/输出/副作用/异常/状态变化约束> | <BI-ID or decision> | <file/function> | <test/manual/static> |

## 当前任务清单

### T1：<任务标题>
- **状态**：`进行中 / 待开始 / 已完成 / 已阻塞`
- **目标**：<当前任务只解决什么>
- **完成标准**：
  - 标准 1
  - 标准 2
- **验证方式**：
  - 验证 1
- **涉及业务约束**：
  - `<BI-ID / none>`
- **涉及函数/方法约束**：
  - `<function/method / none>`
- **文档回写要求**：
  - `<CLAUDE.md / this module doc / both / none>`
- **结果摘要**：`-`
- **最后更新**：`YYYY-MM-DD HH:mm`

### T2：<任务标题>
- **状态**：`待开始`
- **目标**：
- **完成标准**：
- **验证方式**：
- **涉及业务约束**：
- **涉及函数/方法约束**：
- **文档回写要求**：
- **结果摘要**：`-`
- **最后更新**：`-`

## 关键决策

- 决策 1：<为什么这样定>
- 决策 2：<为什么这样定>

## Plan / Implementation Audit

| 时间 | 本轮计划 | 实际实现 | 偏离点 | 新发现规则 | 文档回写位置 | 验证方式 |
|---|---|---|---|---|---|---|
| YYYY-MM-DD HH:mm | <plan summary> | <implementation summary> | <none/details> | <none/details> | <CLAUDE.md/module doc> | <test/manual/static> |

## Diff-to-Doc Cross-check

每轮完成后检查：
- [ ] 新增/修改的业务常量已写回
- [ ] 新增/修改的业务不变量已写回
- [ ] 新增/修改的函数/方法级约束已写回
- [ ] 状态流转、异常路径、权限边界已写回
- [ ] 删除或改变的旧规则已同步更新
- [ ] 用户反馈中的规则已结构化沉淀

## 历史摘要

### YYYY-MM-DD HH:mm
- 本轮完成：
- 改动范围：
- 验证方式：
- 约束变化：
- 文档回写：
- 下一步：

## 使用规则

1. `CLAUDE.md` 是主入口，这个文档只补充复杂模块细节
2. 一次只推进一个当前任务
3. 当前任务边界不清楚时优先提问
4. 如果模块已经不复杂，及时把长期有效内容回收进 `CLAUDE.md`
5. 不要把这里写成另一个大而全总计划
6. 编码前必须完成 plan/code 对账
7. 编码中发现新业务约束必须立即写回
8. 编码后必须记录 implementation-to-plan audit
9. 每轮必须完成 diff-to-doc cross-check
10. 长期有效规则必须同步到 `CLAUDE.md`
