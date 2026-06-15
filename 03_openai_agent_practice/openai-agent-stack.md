# OpenAI Agent Stack

## Mental Model

OpenAI 的 agent 构建可以先按三层理解：

1. `Responses API`：模型调用、工具调用、状态延续、streaming 等核心 API surface。
2. `Agents SDK`：在应用代码里定义 agent、运行 agent loop、管理 handoff、guardrail、approval、MCP 和 tracing。
3. `Evals / Traces / Codex`：用真实运行记录和 eval 驱动后续系统改进。

相关入口：

- Migrate to the Responses API: https://developers.openai.com/api/docs/guides/migrate-to-responses
- Agents SDK overview: https://developers.openai.com/api/docs/guides/agents
- Agent Improvement Loop cookbook: https://developers.openai.com/cookbook/examples/agents_sdk/agent_improvement_loop

## When To Use The Agents SDK Track

优先考虑 Agents SDK，当你的服务端需要自己掌握：

- agent 的 typed application code。
- tool、MCP server、approval、runtime 行为。
- 自定义存储或 conversation strategy。
- 和现有业务逻辑、权限系统、审计系统的深度集成。

官方 Agents SDK overview 将 SDK track 定位为适合服务端拥有 orchestration、tool execution、state 和 approvals 的路径：
https://developers.openai.com/api/docs/guides/agents

## First Agent Checklist

建立第一个 agent 时，不要先做复杂架构。先完成：

- [ ] 一个明确的 `name`。
- [ ] 一段短而具体的 `instructions`。
- [ ] 一个合适的 `model`。
- [ ] 0-3 个必要工具。
- [ ] 明确输出是自然语言还是 structured output。
- [ ] 一个最小 eval 或人工检查样本。
- [ ] 打开 trace 或至少保存运行日志。

官方 Agent definitions:
https://developers.openai.com/api/docs/guides/agents/define-agents

## Runtime Loop

OpenAI Running agents 文档把一次 SDK run 描述为一个 application-level turn：模型被调用，输出被检查，如有工具调用就执行并继续，如有 handoff 就切换 specialist，没有更多工具工作时返回结果。

工程含义：

- runtime loop 是 agent 的核心，不是 prompt 的附属品。
- 工具、handoff、approval、streaming 都应该围绕这个 loop 设计。
- 多轮状态策略必须明确，不能随手混用。

官方 Running agents:
https://developers.openai.com/api/docs/guides/agents/running-agents

## Split Or Not Split

默认先单 agent。满足以下条件时再拆：

- 不同 specialist 需要不同工具面。
- 不同 specialist 需要不同审批或 guardrail。
- 某个分支需要不同模型、输出格式或语气。
- trace 中需要显式显示路由和责任。
- prompt 已经因为职责过多而难以维护。

官方 Orchestration and handoffs:
https://developers.openai.com/api/docs/guides/agents/orchestration

## Open Questions

- 对不同业务场景，什么时候用 Responses API 直接编排，什么时候上 Agents SDK？
- 多 agent 的收益如何通过 eval 证明，而不是靠感觉？
- trace 中哪些字段最值得沉淀为统一日志规范？
