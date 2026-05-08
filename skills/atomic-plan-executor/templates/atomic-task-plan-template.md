# 原子任务卡片（可选）

> legacy / optional reference only。
> 只有当某个复杂模块内部仍然需要进一步细分时，才临时使用这张卡片。
> 默认情况下应直接遵循 `SKILL.md` 的去模板化规则，不要上来就拆成很多原子卡片。

## 任务信息

- 模块：`<module-name>`
- 当前任务：`<task-id or title>`
- 当前状态：`待开始 / 进行中 / 已完成 / 已阻塞`
- 对应计划来源：`<CLAUDE.md section / module doc section>`
- 最近 plan/code 对账：`YYYY-MM-DD HH:mm / none`
- 最后更新：`YYYY-MM-DD HH:mm`

## 本轮目标

这一轮只完成：
- 目标 1

## 本轮涉及约束

- Business Invariants：
  - `<BI-ID / none>`
- Function/Method Constraints：
  - `<function/method constraint / none>`
- 用户反馈规则：
  - `<feedback-derived rule / none>`

## 完成标准

- 标准 1
- 标准 2
- [ ] 本轮新增或改变的业务规则已写回文档
- [ ] 本轮新增或改变的函数/方法约束已写回文档
- [ ] 已完成 implementation-to-plan audit
- [ ] 已完成 diff-to-doc cross-check

## 验证方式

- 验证 1

## 明确不做

- 不做 1
- 不做 2

## 结果摘要

- `-`

## Implementation-to-Plan Audit

- 计划摘要：
- 实际实现：
- 偏离点：`none / details`
- 新发现业务规则：`none / details`
- 文档回写位置：
- 验证方式：

## Diff-to-Doc Cross-check

- [ ] 常量/阈值/默认值已检查
- [ ] 业务不变量已检查
- [ ] 函数/方法行为约束已检查
- [ ] 用户反馈规则已检查
- [ ] 文档已同步或确认无需同步

## 下一步

- `-`

## 使用规则

1. 只有在模块文档仍然不够细时才建立这张卡片
2. 不要默认把每个任务都拆成原子卡片
3. 一旦卡片内容已经稳定，应尽量回收进模块文档或 `CLAUDE.md`
4. 长期规则不要只留在任务卡片里
5. 没有完成 audit 和 diff-to-doc cross-check，不得标记任务完成
6. 如果本轮发现业务分叉或计划缺口，优先提问，不要直接猜测推进
