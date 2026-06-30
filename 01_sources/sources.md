# Sources

资料基线时间：`2026-06-15`。

## OpenAI Official Reading List

### Core Agent Construction

| Priority | Source | Link | Why read |
| --- | --- | --- | --- |
| P0 | Agents SDK overview | https://developers.openai.com/api/docs/guides/agents | 了解 OpenAI 对 SDK-based agent workflow 的整体入口和阅读顺序。 |
| P0 | Agents SDK quickstart | https://developers.openai.com/api/docs/guides/agents/quickstart | 先跑通最小 agent，避免停留在概念层。 |
| P0 | Agent definitions | https://developers.openai.com/api/docs/guides/agents/define-agents | 明确一个 agent 应该包含 model、instructions、tools、guardrails、handoffs、structured outputs 等配置。 |
| P0 | Models and providers | https://developers.openai.com/api/docs/guides/agents/models | 理解 agent 层面的模型选择和运行策略。 |
| P0 | Running agents | https://developers.openai.com/api/docs/guides/agents/running-agents | 理解 agent loop、状态延续和 streaming。 |
| P0 | Orchestration and handoffs | https://developers.openai.com/api/docs/guides/agents/orchestration | 判断什么时候需要多个 agent、handoff 或 agent-as-tool。 |
| P0 | Guardrails and human review | https://developers.openai.com/api/docs/guides/agents/guardrails-approvals | 设计自动检查、人审暂停和敏感动作审批。 |
| P0 | Results and state | https://developers.openai.com/api/docs/guides/agents/results | 理解运行结果、状态和下一轮继续方式。 |
| P0 | Integrations and observability | https://developers.openai.com/api/docs/guides/agents/integrations-observability | 理解 MCP 接入和 tracing/observability。 |
| P0 | Evaluate agent workflows | https://developers.openai.com/api/docs/guides/agent-evals | 把 agent workflow 纳入 eval，而不是只靠主观体验。 |

### Core API Surfaces

| Priority | Source | Link | Why read |
| --- | --- | --- | --- |
| P0 | Migrate to the Responses API | https://developers.openai.com/api/docs/guides/migrate-to-responses | 理解 Responses API 作为现代 agentic workflow 的核心 API surface。 |
| P0 | Using tools | https://developers.openai.com/api/docs/guides/tools | 理解 hosted tools、function tools、MCP 等工具表面的差异。 |
| P0 | Function calling | https://developers.openai.com/api/docs/guides/function-calling | 学会把外部系统能力暴露为清晰、可控的工具接口。 |
| P0 | Structured Outputs | https://developers.openai.com/api/docs/guides/structured-outputs | 用 schema 约束输出，降低解析和下游执行风险。 |
| P1 | Conversation state | https://developers.openai.com/api/docs/guides/conversation-state | 设计多轮 agent 的状态延续策略。 |
| P1 | Prompt engineering | https://developers.openai.com/api/docs/guides/prompt-engineering | 校准 prompt/instructions 的基础写法。 |
| P1 | Reasoning best practices | https://developers.openai.com/api/docs/guides/reasoning-best-practices | 理解 reasoning 模型在复杂任务中的使用边界。 |

### Evaluation, Safety, Production

| Priority | Source | Link | Why read |
| --- | --- | --- | --- |
| P0 | Evaluation best practices | https://developers.openai.com/api/docs/guides/evaluation-best-practices | 建立 eval-driven development 的基本观念。 |
| P0 | Production best practices | https://developers.openai.com/api/docs/guides/production-best-practices | 从原型走向生产时检查密钥、限额、延迟、成本、监控、安全等问题。 |
| P0 | API deployment checklist | https://developers.openai.com/api/docs/guides/deployment-checklist | 上线前的工程检查清单。 |
| P0 | Safety best practices | https://developers.openai.com/api/docs/guides/safety-best-practices | 设计 agent 的安全边界和滥用防护。 |

### Cookbook

| Priority | Source | Link | Why read |
| --- | --- | --- | --- |
| P0 | Build an Agent Improvement Loop with Traces, Evals, and Codex | https://developers.openai.com/cookbook/examples/agents_sdk/agent_improvement_loop | 非常适合作为本项目的核心范例：从 traces 到反馈、eval，再到 Codex 改进。 |
| P1 | Build iterative repair loops with Codex | https://developers.openai.com/cookbook/examples/codex/build_iterative_repair_loops_with_codex | 和 `loop-engineering-lab` 交叉，可用于理解修复循环。 |

## Official Article Notes

| Date | Source | Note | Status |
| --- | --- | --- | --- |
| 2026-06-17 | Build an Agent Improvement Loop with Traces, Evals, and Codex | [openai-agent-improvement-loop.md](openai-agent-improvement-loop.md) | mapped |

## External Sources Queue

后续外部文章登记在这里。登记后不要直接改方法论，先做结构化笔记。

| Date | Type | Title | Link | Status | Notes |
| --- | --- | --- | --- | --- | --- |
| 2026-06-22 | external | Scaling Managed Agents: Decoupling the brain from the hands | https://www.anthropic.com/engineering/managed-agents | mapped | [anthropic-managed-agents.md](anthropic-managed-agents.md) |
|  | external |  |  | queued |  |

## Practice Sources Queue

自己的实践、故障、复盘登记在这里。

| Date | Project | Scenario | Status | Target file |
| --- | --- | --- | --- | --- |
|  |  |  | queued |  |
