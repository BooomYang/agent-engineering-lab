# Guardrails, Evals, Observability

## Guardrails And Human Review

OpenAI Agents SDK 文档将 guardrails 和 human review 放在同一类控制面中：guardrails 做自动检查，human review 在敏感动作前暂停运行并等待审批。

官方入口：
https://developers.openai.com/api/docs/guides/agents/guardrails-approvals

## Control Map

| Risk | First control |
| --- | --- |
| 不应处理的用户请求 | input guardrail |
| 最终输出可能泄漏、违规或不合格 | output guardrail |
| 工具参数危险或不完整 | tool guardrail |
| 写入、删除、取消、发消息、shell、敏感 MCP 动作 | human review |
| 无法判断是否安全 | pause and escalate |

## Human Review Design

人审不是“弹一个确认按钮”就够。审批人需要看到：

- 用户原始意图。
- agent 准备执行的动作。
- 工具名和参数。
- 可能的副作用。
- agent 的理由。
- 可选操作：approve / reject / edit / ask user / escalate。

审批结果需要进入 trace 和下一轮状态，否则 agent 可能重复请求或丢失决策依据。

## Observability

OpenAI Integrations and observability 文档建议在 tuning 前先能 inspect runtime。对 agent engineering 来说，trace 是调试和 eval 的原材料。

官方入口：
https://developers.openai.com/api/docs/guides/agents/integrations-observability

首版 trace 关注：

- agent name。
- model。
- instructions 版本。
- 用户输入。
- context 来源。
- tool calls 和参数。
- tool results。
- handoff。
- guardrail 结果。
- approval 结果。
- final output。
- latency、token、cost。
- error。

## Evals

OpenAI Evaluation best practices 文档强调生成式 AI 有随机性，传统测试不足以覆盖 AI 架构，eval 是衡量质量、可靠性和改进效果的重要方式。

官方入口：

- Agent evals: https://developers.openai.com/api/docs/guides/agent-evals
- Evaluation best practices: https://developers.openai.com/api/docs/guides/evaluation-best-practices

## Eval Design Levels

1. `Final output eval`：最终答案是否正确、有用、安全。
2. `Tool use eval`：是否选对工具、参数是否正确、是否避免不必要工具。
3. `Routing eval`：是否正确 handoff 或调用 specialist。
4. `State eval`：多轮任务是否记住已完成事项和约束。
5. `Safety eval`：是否拒绝、暂停或请求审批。
6. `Cost/latency eval`：是否以可接受成本和延迟完成。

## Agent Improvement Loop

推荐把官方 cookbook 作为精读样本：
https://developers.openai.com/cookbook/examples/agents_sdk/agent_improvement_loop

本项目可采用同样思路：

```text
real traces
  -> human/model feedback
  -> eval cases
  -> issue clustering
  -> system change proposal
  -> Codex implementation or documentation update
  -> rerun evals
```

## Practice Rule

每次修 agent，都尽量留下一个 eval case 或 trace link。没有 eval 的修复，很容易在下一轮 prompt、tool 或模型变更中回退。
