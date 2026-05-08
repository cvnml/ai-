# 原子模块索引（仅复杂模块使用）

> legacy / optional reference only。
> 这个索引不是默认必建，也不再是默认入口。
> 默认情况下应直接遵循 `SKILL.md` 的去模板化规则：根级 `CLAUDE.md` 做项目入口，详细计划写入 owning module 的 `CLAUDE.md`。

## 当前快照

- 激活模块：`<module-name>`
- 模块类型：`<shared / backend / frontend>`
- 当前模式：`<plan-sync / execute / complex-module-assist>`
- 当前任务：`<task-id or title>`
- 上轮完成：`<one-line summary>`
- 下一步：`<one-line next action>`
- 阻塞项：`<none / blockers>`
- 约束同步状态：`<synced / needs-update>`
- 最近审计：`<YYYY-MM-DD HH:mm / none>`
- 最近 diff-to-doc：`<passed / pending / needs-update>`
- 最后更新：`YYYY-MM-DD HH:mm`

## 模块总览

| 模块 | 类型 | 当前状态 | 约束同步 | 最近审计 | 已完成摘要 | 下一步 | 最后更新 |
|---|---|---|---|---|---|---|---|
| <module-name> | <shared/backend/frontend> | <planning/executing/blocked/completed> | <synced/needs-update> | <time/none> | <summary> | <next> | <time> |

## 约束与审计摘要

- Business Invariants：`<none / see module docs / synced to CLAUDE.md>`
- Function/Method Constraints：`<none / see module docs / synced to CLAUDE.md>`
- Plan/Implementation Audit：`<latest summary>`
- Diff-to-Doc Cross-check：`<latest result>`

## 使用规则

1. `CLAUDE.md` 仍然是主入口
2. 这里只追踪复杂模块的执行细节
3. 如果模块已经不复杂，或 `CLAUDE.md` 已足够表达，应删除或停用该索引
4. 不要为了形式完整而给每个项目默认建立该索引
5. 这里只放索引级摘要，详细约束放模块文档，长期规则同步到 `CLAUDE.md`
6. 每轮实现后必须更新最近审计和 diff-to-doc 状态
