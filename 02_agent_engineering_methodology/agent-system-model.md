# Agent System Model

## Minimal Definition

一个可工程化的 agent 不是“模型 + prompt”，而是一个带边界的执行系统：

```text
User goal
  -> agent definition
  -> context/state assembly
  -> model reasoning
  -> tool or handoff decision
  -> external action or observation
  -> guardrail / approval / validation
  -> result, pause, retry, or next turn
  -> trace + eval + improvement loop
```

## Core Components

### 1. Goal Contract

目标契约定义 agent 应该完成什么、不应该做什么、什么时候停止、失败时如何升级。

需要明确：

- 用户任务是什么。
- 任务完成的可观察标准是什么。
- 允许 agent 自主推进到什么程度。
- 哪些动作需要确认或审批。
- 失败、超时、不确定、权限不足时如何处理。

### 2. Agent Definition

agent definition 是一个 specialist 的工程边界。它通常包含：

- name：方便 trace、人审和调试。
- instructions：职责、风格、约束。
- model/settings：能力、延迟、成本和稳定性取舍。
- tools：可调用能力。
- handoffs：何时把控制权交给别的 specialist。
- output contract：最终输出的结构或语义。
- guardrails/approvals：自动检查和人工审批策略。

OpenAI 官方 Agent definitions 文档强调先定义一个清晰、聚焦的 agent，再在确有必要时拆分 specialist：
https://developers.openai.com/api/docs/guides/agents/define-agents

### 3. Tool Surface

工具是 agent 影响外部世界的边界。每个工具都应该被当成一个小型 API 产品来设计，而不是临时函数。

工具规格至少包括：

- name 和 description 是否让模型能正确选择。
- parameters 是否结构化、最小化、带约束。
- return value 是否稳定、可解析。
- side effects 是只读、写入、删除、支付、发消息还是执行代码。
- auth、rate limit、timeout、retry、幂等、审计如何处理。
- 失败时返回什么，是否允许 agent 重试。

相关官方入口：

- Using tools: https://developers.openai.com/api/docs/guides/tools
- Function calling: https://developers.openai.com/api/docs/guides/function-calling

### 4. Context And State

上下文不是越多越好，而是每轮运行时为目标服务的信息包。

需要区分：

- 当前用户输入。
- 会话历史。
- 检索到的知识。
- 工具观察结果。
- 长期记忆或用户偏好。
- 系统策略和权限。
- 已完成步骤和待办状态。

状态策略要回答：下一轮是 replay 历史、使用 session、使用 conversation id，还是使用 previous response id。OpenAI Running agents 文档把状态延续作为 runtime 的核心问题之一：
https://developers.openai.com/api/docs/guides/agents/running-agents

对长任务 agent，还要区分 `session` 和 `context window`：

- `session` 是可持久化、可审计、可恢复的事件日志或状态对象。
- `context window` 是某一轮模型调用可见的工作集。
- harness 可以从 session 中选择、切片、压缩或重组上下文，但这些转换不应该成为唯一事实来源。

Anthropic Managed Agents 文章把 session 设计成 append-only event log，并通过 `getEvents()` 让 harness 选择 event slices。这个模式作为 external source 记录，适合纳入待验证的长任务 runtime 设计假设：
https://www.anthropic.com/engineering/managed-agents

### 5. Orchestration

不要过早多 agent。先问：

- 单个 agent 的职责是否已经过宽？
- 不同分支是否需要不同工具、模型、权限或审批策略？
- 是否需要在 trace 中显式看到路由？
- specialist 是应该接管控制权，还是作为工具被 manager 调用？

官方 Orchestration and handoffs 文档给出的关键方向是：只有当能力隔离、政策隔离、prompt 清晰度或 trace 可读性真的变好时，才增加 specialist：
https://developers.openai.com/api/docs/guides/agents/orchestration

### 6. Guardrails And Human Review

guardrail 是自动检查，人审是暂停和授权机制。二者都不是事后补丁，而应该在设计 agent 的时候进入目标契约。

常见位置：

- input guardrail：阻断不合规或不应处理的请求。
- output guardrail：检查最终输出是否应被改写、隐藏或拒绝。
- tool guardrail：检查工具参数和工具结果。
- human review：对取消订单、发消息、改数据、执行 shell、调用敏感 MCP 等动作暂停审批。

官方入口：
https://developers.openai.com/api/docs/guides/agents/guardrails-approvals

### 7. Observability And Evals

agent 的问题常常不是“最终答案错了”这么简单，而是中间某一步的上下文、工具选择、参数、handoff、审批或恢复策略错了。

因此需要：

- trace：看清每次运行发生了什么。
- logs：记录输入、输出、工具调用、错误和延迟。
- evals：把反复出现的失败变成可回归测试。
- human feedback：校准自动评分是否真的衡量了业务质量。

官方 cookbook 中的 Agent Improvement Loop 给出了很好的方向：从真实 traces 出发，加入人类和模型反馈，转成 eval，再用证据驱动下一轮改进：
https://developers.openai.com/cookbook/examples/agents_sdk/agent_improvement_loop

### 8. Outcome-Guided Improvement Loop

对需要覆盖细节、主观质量或标准一致性的任务，先定义 outcome rubric，再让 agent 产出、独立 grader 评估、agent 根据反馈重试。

关键点：

- rubric 描述“好结果”的可观察标准，而不是只写泛泛目标。
- grader 使用独立上下文，至少能避免完全被 worker 的推理路径牵引。
- feedback 必须指出具体缺口，能变成下一轮修正输入。
- retry 要有次数、成本、风险和人工升级边界。
- 每次尝试、评分、反馈和最终通过理由都要进入 trace。

来自 Anthropic Claude Managed Agents 文章的外部启发：outcomes 用 success rubric 和独立 grader 让 agent 自我修正；dreaming 则把跨 session 的模式提炼进 memory；multiagent orchestration 用 lead/specialist 并行处理复杂任务：
https://claude.com/blog/new-in-claude-managed-agents

在 AI coding 的 loop engineering 里，对应做法是：implementer 负责修改，reviewer/grader 按验收标准、测试结果、diff 范围、风险和文档要求独立检查，再把失败点反馈回实现循环。

这仍是 `待验证` 方法论：rubric 模糊、grader 未校准或重试无上限时，可能只会制造更长的循环，而不是更可靠的结果。

### 9. Versioned Agent Bundle

生产环境里的 agent 行为不是 prompt 单变量，而是一组配置和运行资产的组合。至少包括：

- instructions / prompt。
- model 和 settings。
- tool schema、tool descriptions 和权限策略。
- guardrails、approval policy 和拒绝策略。
- context assembly / retrieval 配置。
- knowledge snapshot 或数据源版本。
- output contract。
- eval baseline 和 release notes。

因此，agent 版本应该被当成一个可审计、可回滚的 bundle，而不是散落在代码、环境变量、数据库和 UI 里的多个隐式状态。

来自 Logic 文章的外部启发：agent production readiness 不只看“回答是否正确”，还要看 typed contracts、testability、version control、observability、model independence 和 robust deployments 是否形成闭环：
https://logic.inc/guides/how-to-build-an-ai-agent

在本项目中，这条先作为 `待验证` 方法论：真实项目里应记录 bundle manifest 是否降低了回滚、复现和对比 eval 的成本。

### 10. Harness Improvement

`harness` 是 agent 周围的完整运行契约，不只是 prompt。它包括：

- instructions 和角色边界。
- tool policy、权限和失败语义。
- routing、handoff 和 control flow。
- output requirements 和 artifact schema。
- validation checks、guardrails 和 human gates。
- trace 字段、eval metadata 和版本信息。

改进 agent 时，先判断问题属于 `missing requirement`、`present but unreliable`，还是 `implementation / observability defect`。这样 Codex 或人类工程师拿到的不是模糊建议，而是可落到 diff、eval 和回滚策略的 harness change。

### 11. Runtime Interfaces

长任务 agent 需要把运行时边界显式化，而不是默认把所有东西放进一个进程或容器：

- brain：模型调用和 harness loop。
- session：持久事件日志、状态和恢复依据。
- hands：sandbox、tools、MCP server、外部系统连接器。
- credential boundary：密钥、OAuth token、repo token 等是否能被 sandbox 或 generated code 读取。
- recovery boundary：brain、session、hand 任一部分失败后，是否能重启、重放、恢复或安全停止。

一个实用原则是：稳定 interface 优先于当前 harness 实现。模型能力、上下文策略和 sandbox 实现会变化，session/event、tool execution、credential proxy、provision/retry 这些接口更应该稳定。

## Design Heuristic

当一个 agent 表现不好时，不要只改 prompt。按这个顺序排查：

1. 目标是否可判定？
2. agent 职责是否过宽？
3. 上下文是否缺失、污染或过载？
4. 工具是否描述不清、参数不稳或失败语义不明确？
5. 是否需要 structured output？
6. 是否需要 guardrail 或人审？
7. trace 是否能解释失败发生在哪里？
8. 是否有 eval 防止下次回退？
9. 当前行为是否能追溯到一个明确的 agent bundle 版本？
10. 是否需要 outcome rubric 和独立 grader 来定义“够好”？
11. 修复目标是否明确落在 harness 的某个部分？
12. session、harness、sandbox/tool 和 credential boundary 是否被错误耦合？
