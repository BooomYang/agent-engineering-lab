# OpenAI Cookbook: Agent Improvement Loop

## Metadata

- Title: Build an Agent Improvement Loop with Traces, Evals, and Codex
- Author: Wesley Pasfield
- Source type: `official`
- Link: https://developers.openai.com/cookbook/examples/agents_sdk/agent_improvement_loop
- Date read: 2026-06-17
- Related project: `agent-engineering-lab`
- Status: `mapped`

Related official docs:

- Agents SDK overview: https://developers.openai.com/api/docs/guides/agents
- Integrations and observability: https://developers.openai.com/api/docs/guides/agents/integrations-observability
- Evaluate agent workflows: https://developers.openai.com/api/docs/guides/agent-evals
- Evaluation best practices: https://developers.openai.com/api/docs/guides/evaluation-best-practices

## One-line Takeaway

这篇 cookbook 把 agent 改进从“修 prompt”推进到一个可运行闭环：真实 traces 产生反馈，反馈转成 eval，eval 结果和 harness 配置再生成 Codex 可执行的改进 handoff。

## What The Source Says

- 示例构建一个基于 OpenAI Agents SDK 的 financial analyst agent，在合成 diligence data 上运行 5 个 traced runs。
- 每个运行都会产出 required artifacts，例如 `summary_answer.md`、`investment_memo.md`、`risk_register.json`、`open_questions.md`、`citations.json` 和 `evidence_table.csv`。
- notebook 明确把 `harness` 定义为模型周围的完整契约，包括 instructions、tools、routing、output requirements 和 validation checks。
- 示例为 agent 定义了 evidence scope、tool policy、unknown-handling、mutation policy 和 eval metadata，使后续优化目标不只限于 prompt wording。
- traces 之后，示例分别加入 human feedback 和 LLM-generated feedback。human feedback 承载领域判断，LLM feedback 扩展行为观察覆盖面。
- 示例用 LLM 根据 traces、human feedback 和 LLM insights 生成 Promptfoo eval definitions，并用 Promptfoo 对当前 trace outputs 执行 validation gate。
- 示例把 harness config、SDK execution traces、human feedback、LLM feedback、generated eval definitions、Promptfoo row results 和 gate summary 写入 HALO optimization context。
- HALO 负责诊断和排序，最终写出 `codex_handoff.md`，交给 Codex 或人类工程师实现下一轮 harness changes。
- 示例区分 reviewed loop 和更自动化的 closed loop。human gates 可以放在 trace review、eval refinement、PR approval、merge 和 deployment 等位置。

## My Interpretation

- `harness` 是本项目需要显式引入的改进对象。agent 质量问题不应默认归因于 prompt，而要定位到 goal contract、instructions、tool policy、routing、output contract、validation、trace 或 eval。
- trace 不是日志归档，而是改进原料。trace 需要能和反馈、eval case、gate result、change proposal 形成可追溯链路。
- human feedback 不只是“主观意见”，它可以结构化为 required observations、prohibited claims、domain invariants 和 eval rubric。
- LLM feedback 适合做覆盖面扩展和候选问题发现，但不能替代 domain expert 对 eval 是否真实有效的校准。
- 自动生成 eval 的价值在于加速从 comment 到 regression case 的转换；长期 suite 仍需要人工收紧、去重、版本化和定期淘汰。
- Codex handoff 应该是 implementation-first artifact，而不是宽泛总结。最小内容包括优先级、证据、预期改动面、验证方式和剩余风险。

## Engineering Surfaces

- [x] Agent definition
- [x] Tool contract
- [x] Context and state
- [x] Orchestration and handoffs
- [x] Guardrails and human review
- [x] Structured output
- [x] Tracing and observability
- [x] Evals
- [x] Production readiness
- [x] Product UX

## Claims To Validate

| Claim | Evidence | How to validate | Status |
| --- | --- | --- | --- |
| 从真实 traces 生成 eval，比纯手写 happy-path eval 更能覆盖 agent 的实际失败模式。 | Cookbook 示例将 traced behavior、human feedback、LLM observations 转成 Promptfoo eval。 | 在真实 agent 项目中比较 trace-derived eval 与人工预设 eval 的回归捕获率。 | 待验证 |
| LLM-generated feedback 能提高问题发现覆盖面，但需要 human calibration。 | 示例把 LLM feedback 和 human feedback 分开，并强调 eval 进入长期 suite 前需要人工检查。 | 抽样评审 LLM feedback 生成的 eval，记录误报、漏报和需要人工重写的比例。 | 待验证 |
| Codex handoff 需要同时携带 ranked changes、supporting evidence 和 validation guidance，才适合进入自动或半自动修复流。 | 示例最终生成 `codex_handoff.md`，并把 HALO diagnosis 和 eval gate result 纳入上下文。 | 在本仓库或真实 agent 项目中试用 handoff 模板，看 Codex 是否能少问澄清并产出可验证改动。 | 待验证 |
| reviewed loop 可以作为 closed loop 的前置形态，只有当 eval gate 和审批边界足够可信后再提高自动化程度。 | Cookbook 明确区分 reviewed loop 和 closed loop，并允许在多处加入 human gates。 | 记录不同自动化级别下的误改、漏改、人工审查耗时和回滚次数。 | 待验证 |

## What To Update

- Methodology: 在 agent system model 中明确 `harness` 是改进对象；trace、feedback、eval、handoff 是同一条改进链。
- OpenAI practice: 扩展 `guardrails-evals-observability.md` 的 Agent Improvement Loop 部分，补上 trace-to-eval 和 reviewed/closed loop 设计。
- Template: 新增 `04_templates/trace-to-eval-handoff.md`。
- Case: 暂无真实项目案例，后续可把第一个 agent 改进循环写入 `05_cases/`。
- Experiment: 可设计一个小实验，比较 trace-derived eval 与人工预设 eval。
- Framework backlog: 将 Trace-to-eval 模板标为已落地，并新增 handoff schema/checker 方向。

## Open Questions

- 本项目应该采用 OpenAI trace grading、Promptfoo、自研 eval runner，还是保留多 runner 适配层？
- `harness` 变更如何版本化，才能把 trace、eval result 和代码 diff 对上？
- 哪些 human gates 必须保留，哪些可以在 eval gate 稳定后自动化？
- HALO 是当前 cookbook 的一个优化工具选择，是否应作为本项目长期方法论的一部分，需要通过实践再判断。
