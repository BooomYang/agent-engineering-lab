# Agent Engineering Methodology

这里沉淀工具无关的方法论：无论使用 OpenAI Agents SDK、Responses API、其他 agent framework，还是自研 runtime，都应该能复用。

## Current Model

一个 agent system 至少由九个工程面组成：

1. `Goal`：它要替用户完成什么任务，停止条件是什么。
2. `Agent definition`：它的职责、指令、模型、输出契约和可调用能力。
3. `Tools`：它能改变或查询外部世界的接口，以及每个接口的权限、输入、输出和失败语义。
4. `Context and state`：它在每轮运行中看到什么，跨轮如何延续，长期记忆如何进入上下文。
5. `Orchestration`：单 agent、多 agent、handoff、agent-as-tool 或 workflow 的分工方式。
6. `Guardrails and approvals`：什么可以自动做，什么必须阻断、降级或让人审批。
7. `Runtime loop`：执行、观察、工具调用、继续、暂停、恢复、终止的控制流。
8. `Observability and evals`：用 traces、logs、metrics、evals 和人工反馈判断 agent 是否变好。
9. `Production operations`：密钥、权限、成本、延迟、限额、监控、安全、版本和发布策略。

## Files

- [agent-system-model.md](agent-system-model.md)：agent system 的基础结构图。
- [principles.md](principles.md)：首版工程原则。
- [risk-map.md](risk-map.md)：agent 常见风险地图。
- [glossary.md](glossary.md)：术语表。

## Working Rule

每一条新方法论都要回答三个问题：

1. 它解决哪个 agent 工程面的什么问题？
2. 它来自官方文档、外部来源、实践观察还是个人假设？
3. 它如何被真实项目、eval 或 trace 验证？
