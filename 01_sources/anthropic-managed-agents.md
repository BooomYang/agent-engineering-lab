# Anthropic Engineering: Managed Agents

## Metadata

- Title: Scaling Managed Agents: Decoupling the brain from the hands
- Author: Lance Martin, Gabe Cemaj, Michael Cohen
- Source type: `external`
- Link: https://www.anthropic.com/engineering/managed-agents
- Date read: 2026-06-22
- Related project: `agent-engineering-lab`
- Status: `mapped`

## One-line Takeaway

长任务 agent 的运行时不应该把模型 loop、session log 和 sandbox 执行环境绑成一个不可替换的整体；稳定接口比当前 harness 形态更重要。

## What The Source Says

- Anthropic 介绍了 Claude Managed Agents 的架构思路：用一组稳定接口承载 long-horizon agent work，使底层 harness、session 和 sandbox 实现可以变化。
- 文章把 agent 运行时拆成三个虚拟化组件：`session` 是 append-only event log，`harness` 是调用模型并路由 tool calls 的 loop，`sandbox` 是运行代码和编辑文件的执行环境。
- 早期把 session、harness、sandbox 放在同一个 container 内，带来直接文件 syscall 和较少服务边界，但也导致 container failure 会丢 session、debug 依赖进容器、以及用户数据和调试环境耦合。
- 文章提出将 brain，也就是模型和 harness，与 hands，也就是 sandboxes 和 tools，以及 session log 解耦。sandbox 以 tool interface 暴露，例如 `execute(name, input) -> string`；失败可作为 tool-call error 返回，必要时重新 provision。
- harness 也被设计成可替换、可恢复的无状态组件。session log 位于 harness 外部，新的 harness 可通过 session id 读取 event log 并恢复。
- 安全边界上，文章强调不要让 generated code 所在 sandbox 接触 credentials。Git token 可以在 sandbox 初始化时绑定到 repo remote；自定义工具通过 MCP proxy 和 vault 取 credential，harness 本身也不持有凭据。
- 文章区分 session 和模型 context window。session 是持久、可查询的 event log；harness 可以按需选择、切片、转换 event，再放入模型 context window。
- 解耦后，container 只在需要时作为 tool provision，降低 time-to-first-token。文章报告该架构下 p50 TTFT 下降约 60%，p95 下降超过 90%。
- 文章把 Managed Agents 称为 meta-harness：对 Claude 周围的 interface 有明确意见，但不锁定未来具体 harness、sandbox 或 tool 实现。

## My Interpretation

- 这篇文章强化了本项目已有的 `harness` 概念：harness 不只是 prompt 容器，而是运行时策略层。它可以变化，所以应该被放在稳定 interface 之后。
- `session != context window` 是关键原则。session 应该作为可恢复、可查询、可审计的事实日志；context window 只是当前 turn 的工作集。
- sandbox 和 tool 应该按 capability interface 设计，不能假设所有资源都在同一台机器、同一容器或同一权限域。
- 长任务 agent 的可靠性来自可恢复边界：session 可持久化，harness 可重启，sandbox 可重新 provision，tool failure 可回到 agent loop。
- 安全不应只依赖“限制模型不会做某事”。更稳的结构是让 sandbox 无法接触 credential，credential 由外部 vault、proxy 或资源绑定机制代理。
- 这篇是 Anthropic 产品和架构经验，不能直接变成所有 agent 系统的规则；但它提供了一个值得验证的运行时架构假设：stable runtime interfaces 可以降低模型和 harness 演进带来的重构成本。

## Engineering Surfaces

- [x] Agent definition
- [x] Tool contract
- [x] Context and state
- [x] Orchestration and handoffs
- [x] Guardrails and human review
- [ ] Structured output
- [x] Tracing and observability
- [ ] Evals
- [x] Production readiness
- [x] Product UX

## Claims To Validate

| Claim | Evidence | How to validate | Status |
| --- | --- | --- | --- |
| 将 session log 放在 harness 外部，可以提升长任务 agent 的崩溃恢复能力。 | Anthropic Managed Agents 架构把 session 作为持久 event log，harness 通过 session id 恢复。 | 在真实 agent 项目中注入 harness crash，验证是否能从 event log 恢复且不重复危险副作用。 | 待验证 |
| 将 sandbox 作为 tool 而不是 harness 所在环境，可以降低容器失败对 agent session 的影响。 | 文章描述 sandbox 失败被作为 tool-call error 返回，并可重新 provision。 | 对比单容器 harness 和解耦 sandbox 在 container failure、timeout、cold start 下的恢复率。 | 待验证 |
| credential 不进入 sandbox，比单纯缩小 token 权限更稳健。 | 文章把 Git token 绑定到 remote，MCP OAuth token 存在 vault 并通过 proxy 调用。 | 对生成代码执行环境做注入测试，验证 sandbox 无法读取长期 credential。 | 待验证 |
| session 作为可查询对象而不是直接塞入 context window，有助于降低不可逆 compaction/trim 带来的信息丢失。 | 文章提供 `getEvents()` 读取 event slices 的模式，并将 context management 留给 harness。 | 在长任务中比较直接压缩上下文、文件记忆、event log 查询三种策略的失败模式。 | 待验证 |
| brain/hands 解耦可以改善 TTFT，但收益取决于任务是否经常不需要 sandbox。 | 文章报告 p50 TTFT 下降约 60%、p95 下降超过 90%。 | 在本项目未来 starter kit 中记录 cold start、tool provision、first token 和 first action latency。 | 待验证 |

## What To Update

- Methodology: 在 agent system model 中加入 `Runtime Interfaces`，把 session、harness、sandbox/tool、credential boundary 和 recovery boundary 显式化。
- Risk map: 增加 coupled runtime、session loss、credential reachable from sandbox、context window 和 session 混淆等风险。
- Template: 更新 `agent-design-brief.md`，加入 Runtime Architecture 检查项。
- Framework backlog: 增加 managed-agent runtime interface checklist。

## Open Questions

- 对中小型 agent 项目，何时需要完整 brain/hands/session 解耦，何时简单本地 runtime 已足够？
- session event log 应该记录到什么粒度，才能兼顾恢复、审计、隐私和成本？
- 如果 sandbox 失败后重新 provision，哪些副作用必须用幂等键或事务日志防重复？
- 多个 brains 共享或传递 hands 时，权限、审计和责任边界如何表达？
