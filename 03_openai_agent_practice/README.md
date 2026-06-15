# OpenAI Agent Practice

这里沉淀基于 OpenAI 官方能力的 agent 构建实践。和 `02_agent_engineering_methodology` 的区别是：这里可以明确引用 OpenAI API、Agents SDK、Responses API、工具、guardrails、traces、evals 等具体产品能力。

## Files

- [openai-agent-stack.md](openai-agent-stack.md)：OpenAI agent 技术栈的首版地图。
- [tool-and-context-design.md](tool-and-context-design.md)：工具、MCP、function calling、上下文和状态设计。
- [guardrails-evals-observability.md](guardrails-evals-observability.md)：guardrails、人审、traces、evals 的实践路径。
- [production-readiness.md](production-readiness.md)：上线前的工程检查。

## Official Baseline

首版重点阅读：

- Agents SDK overview: https://developers.openai.com/api/docs/guides/agents
- Responses API migration guide: https://developers.openai.com/api/docs/guides/migrate-to-responses
- Using tools: https://developers.openai.com/api/docs/guides/tools
- Agent evals: https://developers.openai.com/api/docs/guides/agent-evals
- Agent improvement loop cookbook: https://developers.openai.com/cookbook/examples/agents_sdk/agent_improvement_loop
