# Tool And Context Design

## Tool Design

OpenAI 官方资料中，工具调用是 agentic workflow 的核心表面之一。对工程实践来说，重点不是“能不能调用函数”，而是“工具是否被设计成模型可理解、系统可控制、结果可验证的契约”。

官方入口：

- Using tools: https://developers.openai.com/api/docs/guides/tools
- Function calling: https://developers.openai.com/api/docs/guides/function-calling
- Structured Outputs: https://developers.openai.com/api/docs/guides/structured-outputs

## Function Tool Checklist

每个 function tool 至少要检查：

- [ ] `name` 是否短、稳定、动作明确。
- [ ] `description` 是否说明何时使用，而不只是复述函数名。
- [ ] 参数是否使用 schema 严格约束。
- [ ] 参数是否最小化，避免模型填太多自由文本。
- [ ] 返回值是否稳定、简洁、可解析。
- [ ] 错误是否是 agent 可理解的业务错误，而不是裸异常。
- [ ] 是否标注副作用等级：read / write / delete / external-send / code-exec / payment。
- [ ] 高风险副作用是否需要 approval。

## Structured Output

只要输出会被程序继续处理，就优先用 structured output。

典型场景：

- 抽取信息。
- 生成计划。
- 选择路由。
- 生成工具参数草稿。
- 生成 UI 状态。
- 生成 eval 评分和理由。

OpenAI Structured Outputs 文档强调可以用 JSON schema 约束模型输出，并建议 key 命名清晰、重要字段提供 description、通过 eval 判断结构是否适合用例：
https://developers.openai.com/api/docs/guides/structured-outputs

## Context Strategy

设计上下文时先回答：

- 本轮任务需要哪些事实？
- 哪些历史必须保留，哪些应该摘要或丢弃？
- 哪些信息来自用户，哪些来自工具，哪些来自系统策略？
- 哪些信息有时效性？
- 哪些信息不应该进入模型上下文？

## State Strategy

OpenAI Running agents 文档列出多种状态延续方式，包括 application history、session、conversation id、previous response id。工程上应该为每个 conversation 选择一种主策略。

不要随意混用，否则容易出现：

- 历史重复。
- 工具结果重复进入上下文。
- 用户意图被旧消息污染。
- agent 不知道哪些步骤已经完成。

官方 Running agents:
https://developers.openai.com/api/docs/guides/agents/running-agents

官方 Conversation state:
https://developers.openai.com/api/docs/guides/conversation-state

## MCP

MCP 适合把外部系统能力接入 agent。设计时要区分：

- hosted/remote MCP：模型通过 hosted surface 调远端 MCP。
- local/private MCP：应用 runtime 自己连接私有或本地 MCP，自己控制网络、权限和审批。

官方 Integrations and observability:
https://developers.openai.com/api/docs/guides/agents/integrations-observability

## Practice Hypotheses

- 一个工具的 description 应该同时服务于模型选择、审计阅读和未来 eval。
- 工具返回值越接近“业务事实”，agent 后续推理越稳定。
- 对写操作，先让 agent 生成 action proposal，再通过 human review 或 policy checker 执行，通常比直接执行更稳。
