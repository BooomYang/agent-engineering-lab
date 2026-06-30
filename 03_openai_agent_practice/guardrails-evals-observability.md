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

## Trace-To-Eval Intake

从一次 agent 运行到一个可回归的 eval，至少要保留这条证据链：

| Signal | Purpose |
| --- | --- |
| Trace | 记录模型调用、工具调用、handoff、guardrail、输出 artifact 和错误。 |
| Human feedback | 标记领域上必须出现的观察、禁止出现的结论、风险优先级和业务判断。 |
| Model feedback | 扩展可疑行为、重复模式和可能缺失的检查点，但不替代人工校准。 |
| Eval definition | 把反馈变成 expected behavior、rubric、deterministic assertion、pass/fail example。 |
| Gate result | 记录当前 harness 是否通过，以及失败解释。 |
| Handoff | 把证据、优先级、改动面和验证方式交给 Codex 或人类工程师。 |

OpenAI 当前 Agents SDK 文档将 traces 放在 debugging 和系统化评估之间：先用 traces 看清行为，再用 graders、datasets 和 eval runs 做重复评估。

相关入口：

- Integrations and observability: https://developers.openai.com/api/docs/guides/agents/integrations-observability
- Evaluate agent workflows: https://developers.openai.com/api/docs/guides/agent-evals

## Harness Change Handoff

Codex handoff 不应该只是总结，而应该能直接进入实现工作。最小字段：

- 当前 harness version 和相关 trace ids。
- 反馈来源：human feedback、model feedback、eval result、runtime error。
- 问题分类：`missing requirement` / `present but unreliable` / `implementation or observability defect`。
- 目标改动面：prompt、tool policy、tool implementation、routing、output schema、validator、eval suite、human gate。
- 推荐优先级、影响、置信度和实现成本。
- 验证方式：要新增或重跑哪些 eval，是否需要人工复核。
- 剩余风险和回滚方式。

## Reviewed Loop Before Closed Loop

默认先用 reviewed loop：

```text
trace + feedback
  -> generated eval proposal
  -> human eval refinement
  -> Codex handoff
  -> reviewed PR
  -> rerun eval gate
  -> promote or rollback
```

只有当 eval gate、artifact validation、权限边界和回滚策略都稳定后，才考虑更自动化的 closed loop。即使进入 closed loop，human gates 仍然可以保留在 trace review、eval refinement、PR approval、merge 或 deployment 上。

注意：OpenAI Evaluation best practices 在 2026-06-17 已提示 legacy Evals platform 有退役时间表。本项目沉淀的是 eval-driven improvement 的工程原则和 agent workflow evaluation surface，不把任何单一 eval 平台写成长期默认。

## Practice Rule

每次修 agent，都尽量留下一个 eval case 或 trace link。没有 eval 的修复，很容易在下一轮 prompt、tool 或模型变更中回退。
