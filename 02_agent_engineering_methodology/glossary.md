# Glossary

## Agent

一个具有目标、指令、工具、上下文、运行循环和边界控制的 AI 执行单元。

## Agent Definition

定义一个 agent 的工程契约，包括 name、instructions、model、tools、handoffs、output type、guardrails 等。

## Tool

agent 可调用的外部能力，可以是函数、本地服务、MCP server、hosted tool、检索系统或业务 API。

## Tool Contract

工具暴露给 agent 的稳定接口，包括名称、描述、参数、返回值、权限、副作用、失败语义和审计要求。

## Context

一次运行中提供给模型的信息集合，包括用户输入、历史、检索结果、工具观察、策略、状态和记忆。

## State

跨轮保留的信息和运行位置。可能由应用、本地 session、server-managed conversation 或 previous response id 管理。

## Handoff

一个 agent 把控制权交给另一个 specialist 的机制。

## Agent As Tool

一个 specialist 不接管控制权，而是被 manager 当作工具调用。

## Guardrail

自动检查机制，可用于输入、输出或工具调用周边。

## Human Review

运行暂停，让人或外部策略审批敏感动作，再继续、拒绝或改写。

## Trace

一次或一组 agent run 的执行记录，用于调试 prompt、工具、handoff、approval 和状态。

## Eval

结构化测试，用来衡量 agent 或 LLM 应用在特定任务分布上的质量、可靠性、安全性或性能。

## Agent Improvement Loop

从 traces 和真实反馈出发，形成 eval，定位问题，修改系统，再回归验证的持续改进循环。
