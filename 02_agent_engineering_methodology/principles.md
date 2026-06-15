# Principles

## P1. Start With One Focused Agent

先做一个职责清晰的 agent，再考虑多 agent。

拆分 agent 的合理理由是：工具面不同、审批策略不同、模型或输出风格不同、职责边界不同、trace 中需要显式路由。不要因为“多 agent 听起来更强”而拆分。

## P2. Treat Tools As Product APIs

工具不是函数列表，而是 agent 与外部世界的契约。工具名称、描述、参数、返回值、失败语义、权限和审计都会直接影响 agent 行为。

一个工具如果人类工程师都难以理解，模型也不可能稳定使用。

## P3. Separate Reasoning From Authority

模型可以判断需要做什么，但不代表它应该拥有执行所有动作的权限。

只读查询、草稿生成、写入数据、发消息、删数据、付款、执行代码、调用私有系统，应该有不同权限和审批策略。

## P4. State Is A Design Choice

多轮 agent 的状态延续不能随意混用。要明确是本地 replay、session、server-managed conversation，还是 previous response id。

状态混乱会导致上下文重复、历史污染、遗漏已完成步骤、权限边界不清。

## P5. Structured Output Is A Reliability Tool

只要输出会进入下游程序、数据库、工具调用、UI 状态或 eval，就应该优先考虑结构化输出。

自然语言适合沟通，schema 适合执行。

## P6. Guardrails Belong In The Architecture

guardrail 和 human review 不应该等事故后再加。只要 agent 能触发外部副作用，就应该在设计阶段定义哪些动作自动执行、哪些动作暂停审批、哪些动作直接拒绝。

## P7. No Trace, No Debuggability

agent 的错误经常发生在中间步骤。没有 trace，就只能猜 prompt、工具、上下文还是模型出了问题。

trace 是 eval 和改进循环的原材料。

## P8. Eval From Real Failures

eval 不应该只来自想象中的 happy path。真实用户失败、工具异常、误路由、拒答、幻觉、越权尝试、上下文遗漏，都应该沉淀为 eval case。

## P9. Production Is A Separate Discipline

能跑 demo 不等于能上线。生产化至少要处理密钥、限额、成本、延迟、监控、日志、数据安全、版本、回滚和人工运营。

OpenAI Production best practices 可作为基础检查来源：
https://developers.openai.com/api/docs/guides/production-best-practices

## P10. Improve The System, Not Just The Prompt

agent 质量问题可能来自 prompt，但也可能来自目标定义、工具契约、上下文、状态、审批、eval、trace 或产品交互。优先定位问题所在的工程面，再选择改法。
