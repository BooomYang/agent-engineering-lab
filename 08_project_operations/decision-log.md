# Decision Log

## 2026-06-15: Create `agent-engineering-lab` As A Separate Project

Decision:

创建 `agent-engineering-lab`，与 `loop-engineering-lab` 并列。

Reasoning:

- `loop-engineering-lab` 聚焦反馈循环和 AI coding agent 的执行/验证/修正范式。
- `agent-engineering-lab` 聚焦完整 agent 系统的设计、构建、评估、上线和迭代。
- loop engineering 可以作为 agent engineering 的关键方法之一，但不是全部。

Implication:

- 两个项目互相引用，但各自维护 README、sources、methodology、templates、cases、experiments、operations。
- agent 相关官方文档、外部文章和实践复盘优先进入本项目。
- AI coding loop、Codex 工作流和修复循环仍优先沉淀到 `loop-engineering-lab`。
