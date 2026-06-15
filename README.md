# Agent Engineering Lab

这是一个用于沉淀 agent engineering 工程方法论与实践的长期工作区。

首版目标不是写完整理论，而是搭出一个能持续迭代的骨架：官方文档能进入来源库，外部技术文章能被结构化吸收，真实 agent 项目能进入案例库，反复出现的问题能沉淀为原则、模板、eval、guardrail、工具契约或未来框架能力。

## Working Definition

这里暂定的 `agent engineering` 是：围绕一个能感知目标、调用工具、维护上下文、执行多步任务、接受约束并被持续评估的 AI agent，设计其产品形态、系统架构、运行边界、反馈机制和工程运营方式。

它不是简单地“给模型加几个工具”，而是一个完整系统工程：

- 从 prompt 设计，转向定义 agent 的职责、输入输出、工具边界、状态策略和失败出口。
- 从单次模型调用，转向可观察、可恢复、可评估的 agent runtime。
- 从依赖模型聪明程度，转向设计上下文、工具、guardrail、人审、eval、trace 和生产监控。
- 从一次性 demo，转向用真实 traces、失败案例、用户反馈和 eval 数据持续改进 agent。

## Boundary With Loop Engineering

本项目和 `loop-engineering-lab` 并列，而不是上下级替代。

- `loop-engineering-lab` 关注：如何设计可靠的反馈循环，让 AI coding agent 在任务中持续推进、验证和修正。
- `agent-engineering-lab` 关注：如何构建一个完整、可用、可评估、可上线、可迭代的 agent 系统。

换句话说，loop engineering 是 agent engineering 中非常重要的一类执行与改进方法；agent engineering 的范围还包括 agent 定义、工具设计、上下文/记忆、权限、人机协作、可观测性、产品体验和生产治理。

## Source Baseline

首版内容基于 OpenAI 官方公开文档和 cookbook，资料基线时间为 `2026-06-15`。官方链接集中登记在 [sources.md](01_sources/sources.md)。

外部文章和个人实践可以进入本项目，但必须标注来源类型：

- `official`：OpenAI 官方文档、官方 cookbook、官方博客。
- `external`：第三方文章、论文、开源项目、演讲。
- `practice`：自己的项目经验、调试记录、复盘。
- `hypothesis`：尚未经过实践验证的判断。

## Directory Map

- [01_sources](01_sources/README.md)：资料来源、精读清单、文章笔记模板。
- [02_agent_engineering_methodology](02_agent_engineering_methodology/README.md)：工具无关的方法论、系统模型、原则和风险地图。
- [03_openai_agent_practice](03_openai_agent_practice/README.md)：基于 OpenAI 官方能力的 agent 构建实践。
- [04_templates](04_templates/README.md)：agent 设计、工具规格、eval 计划、实践复盘模板。
- [05_cases](05_cases/README.md)：真实 agent 项目案例与复盘。
- [06_experiments](06_experiments/README.md)：小实验、对照验证、失败样本。
- [07_framework_roadmap](07_framework_roadmap/README.md)：未来框架化能力地图与 backlog。
- [08_project_operations](08_project_operations/README.md)：本工作区自身的运行循环、决策记录和新会话入口。
- [.agents/skills/agent-engineering-research](.agents/skills/agent-engineering-research/SKILL.md)：用于吸收新资料和实践反馈的 Codex repo skill。

## Iteration Loop

1. 记录来源：把官方文档、技术文章、项目反馈或失败样本登记到 `01_sources/sources.md` 或 `05_cases/`。
2. 归类问题：判断它属于 agent 定义、工具、上下文/状态、orchestration、guardrail、人审、eval、trace、部署还是产品体验。
3. 提炼主张：区分事实、作者观点、自己的推断和待验证假设。
4. 分流沉淀：通用结论进入 `02_agent_engineering_methodology/`，OpenAI 专用实践进入 `03_openai_agent_practice/`。
5. 固化资产：能复用的内容优先沉淀为 `04_templates/`、checklist、eval、tool spec 或 `.agents/skills/`。
6. 实战验证：在真实 agent 项目中试用，并记录成功、失败、成本、延迟、质量和上下文条件。
7. 复盘升级：反复出现的问题升级为 guardrail、human review、trace 规范、eval 集或未来框架 backlog。

## Current Priorities

- P0：建立 agent engineering 的基础结构图，避免把 agent 简化成“prompt + tools”。
- P0：把 OpenAI 官方 agent 相关文档转化为可执行的工程检查项。
- P0：为后续外部文章和个人实践准备稳定吸收路径。
- P1：沉淀 agent 定义、工具契约、状态策略、guardrail、eval、observability 的模板。
- P1：用真实项目复盘反推方法论，而不是只做概念整理。
- P2：等实践稳定后，再考虑沉淀为框架、CLI、插件、starter kit 或自动化能力。

## Using This With Codex

在这个目录内使用 Codex 时，优先读取 [AGENTS.md](AGENTS.md)。当你要吸收新文章、整理实践复盘或更新方法论时，可以显式调用：

```text
$agent-engineering-research
```

OpenAI 相关结论优先以官方文档为准。外部文章先作为研究来源，不直接变成规则，除非经过实战验证或明确标注为待验证假设。

新开会话时，从 [New Session Brief](08_project_operations/new-session-brief.md) 恢复上下文。
