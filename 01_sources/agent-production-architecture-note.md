# Article Note: 如何让 AI 快速搭建一套生产 Agent？

## Metadata

- Title: 如何让 AI 快速搭建一套生产 Agent？全面理解 Agent 架构
- Author: 未标注
- Source type: `external`
- Link: [local article](如何让%20AI%20快速搭建一套生产%20Agent%20？全面理解%20Agent%20架构.md)
- Date read: 2026-06-30
- Related project: `agent-engineering-lab`
- Status: `mapped`

## One-line Takeaway

这篇文章的价值不在深度，而在把生产 agent 常见失效面用通俗清单串起来；吸收时要把“全量清单”重新拆成最小必需边界和按风险引入的进阶模块。

## What The Source Says

- agent 不是简单的模型 harness，而是围绕目标、状态、工具、记忆、检索、权限、评估和恢复机制构建的自动化系统。
- 判断一个 agent 方案是否靠谱，可以看状态、workflow、工具 schema/权限、RAG、memory、防 prompt injection、trace、eval 和失败恢复。
- RAG 不应只做向量检索；文章建议 hybrid search、metadata filter、query rewrite、rerank、chunk/source metadata。
- 复杂任务应显式设计状态流转、workflow、Planner/Executor/Verifier 职责边界，而不是依赖一个大 prompt。
- 工具需要 schema、错误格式、权限等级、副作用说明和 human approval。
- 生产化需要 guardrails、human-in-the-loop、trace、eval、retry/recovery、幂等、sandbox、output contract、cost control 和 caching。

## My Interpretation

文章方向和本项目现有 `Agent System Model` 一致：agent engineering 的关键是把模型自由度放进目标、工具、上下文、权限、观测和评估边界里。

需要校正的是“最小可用”边界。文章把 RAG、reranker、memory 分层、FSM、Planner/Executor/Verifier、sandbox、caching 等都放进一个生产清单，适合提醒开发者别漏风险面，但不适合作为所有 agent 的强制骨架。更稳妥的吸收方式是：

- 把 goal contract、agent definition、context/state、tool contract、safety controls、trace、eval/recovery 视为生产级 agent 的最小边界。
- 把 RAG、长期 memory、显式 FSM/workflow engine、multi-agent、sandbox、caching、cost optimizer 等视为按场景触发的进阶能力。
- 每个进阶模块都要有引入条件，否则会增加状态、trace、eval 和运维复杂度。

## Engineering Surfaces

- [x] Agent definition
- [x] Tool contract
- [x] Context and state
- [x] Orchestration and handoffs
- [x] Guardrails and human review
- [x] Structured output
- [x] Tracing and observability
- [x] Evals
- [x] Production readiness
- [x] Product UX

## Claims To Validate

| Claim | Evidence | How to validate | Status |
| --- | --- | --- | --- |
| Hybrid search + rerank 是“生产 RAG”的默认方案 | 文章观点；与常见 RAG 实践一致，但不是每个 agent 都需要 RAG | 在真实知识库 agent 中对比 vector-only、hybrid、hybrid+rerank 的命中率、答案正确率、延迟和成本 | 待验证 |
| FSM 是 agent 实现的必需品 | 文章观点；本项目更倾向于“显式状态策略是必需，FSM 是一种实现” | 对长流程任务比较轻量 state record、FSM、workflow engine 的复杂度和恢复能力 | 待验证 |
| Planner/Executor/Verifier 应作为默认架构 | 文章观点；本项目更倾向于复杂任务再引入 | 在代码审查、调研、写作等任务中比较单 agent 分步骤 vs P/E/V 的质量、延迟和调试成本 | 待验证 |
| 失败恢复比成功路径更重要 | 与本项目 `risk-map`、`production-readiness` 一致 | 从真实失败 trace 中统计缺少恢复策略导致的中断比例 | 待验证 |

## What To Update

- Methodology: 在 `02_agent_engineering_methodology/agent-system-model.md` 增加最小骨架和进阶模块边界。
- OpenAI practice: 暂不更新；现有 guardrails/evals/observability、tool/context、production readiness 已覆盖主要方向。
- Template: 后续可把边界表加入 `04_templates/agent-design-brief.md`。
- Case: 等真实 agent 项目验证后再沉淀。
- Experiment: 可设计 RAG 和 workflow/state 对照实验。
- Framework backlog: 如未来做 starter kit，可把“能力按触发条件启用”作为框架原则。

## Open Questions

- 对不同 agent 类型，最小 eval 的样本量和覆盖面应该如何设定？
- 什么时候轻量状态记录已经足够，什么时候必须升级为 FSM 或 workflow engine？
- RAG pipeline 的默认复杂度应该如何随知识库规模、召回风险和延迟预算变化？
