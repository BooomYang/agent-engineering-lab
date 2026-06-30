# Risk Map

## Goal Risks

- 目标不可判定：没有明确完成标准。
- 停止条件模糊：agent 反复执行或过早结束。
- 任务边界过宽：一个 agent 同时承担规划、执行、审核、沟通、审批。

## Context Risks

- 上下文缺失：关键业务规则、用户偏好、历史状态没有进入运行。
- 上下文污染：无关历史、过期信息、错误工具结果影响判断。
- 状态重复：本地历史和 server-managed state 混用导致信息重复。
- 长期记忆越权：把不该持久化的信息带入后续任务。
- 混淆 session 和 context window：把当前模型可见上下文当成唯一事实来源，导致 compaction、trim 或 crash 后不可恢复。
- session log 不可查询：trace 有记录但 agent runtime 无法按需读取、切片或恢复。
- 记忆沉淀污染：把一次性 workaround、错误推断或过期偏好写入长期记忆。
- 团队记忆越界：把个人偏好误当成团队规范，或把项目规则带到无关项目。

## Tool Risks

- 工具描述不清：模型不知道何时调用。
- 参数过宽：模型能传入危险、无效或歧义参数。
- 返回值不稳定：下游解析困难。
- 副作用不可控：写入、删除、付款、发消息没有审批或审计。
- 失败不可恢复：工具报错没有可执行的下一步。
- sandbox 和工具环境耦合过深：执行环境失败时，agent session、状态和调试能力一起丢失。
- credential 可被 generated code 读取：sandbox 内能看到长期 token、环境变量或外部系统凭据。

## Orchestration Risks

- 过早拆分：多 agent 增加 prompt、trace、审批和状态复杂度。
- 路由错误：handoff 条件不清导致 specialist 被误用。
- 责任漂移：manager 和 specialist 都以为对方负责最终质量。
- nested loop 失控：agent-as-tool 内部失败但外层不可见。
- 并行冲突：多个 specialist 共享文件系统或上下文时互相覆盖结果。
- 汇总污染：lead agent 把低质量 specialist 输出直接合并，缺少证据和置信度标注。

## Output Risks

- 自然语言被程序解析：高概率产生边界错误。
- schema 太松：下游仍要猜字段含义。
- schema 太复杂：模型遵循难度高，eval 和调试成本上升。
- refusal 未处理：安全拒答不符合业务 schema 时下游崩溃。

## Safety And Approval Risks

- 敏感动作自动执行。
- guardrail 只检查输入，不检查输出和工具参数。
- 人审没有足够上下文，审批人无法判断。
- 审批结果没有回写到 agent 状态。

## Evaluation Risks

- 只测最终答案，不测工具选择、参数、状态和审批。
- 只测 happy path，不测真实失败。
- 自动评分没有经过人工校准。
- eval 数据和生产分布脱节。
- outcome rubric 模糊：grader 只能给泛泛反馈，无法驱动下一轮修正。
- worker 和 grader 上下文未隔离：grader 被 worker 的解释带偏，而不是独立检查结果。
- 自我修正无上限：agent 反复重试，成本和延迟失控。

## Production Risks

- API key 泄漏或权限过大。
- 没有 rate limit 和成本上限。
- 没有 trace/log，线上问题无法定位。
- 没有版本和回滚策略。
- 没有数据保留、隐私和合规边界。
- harness、session、sandbox 同故障域：单个容器或进程失败会丢失长任务上下文。
- 缺少可恢复边界：harness crash、tool timeout、sandbox restart 后无法判断是否应该 retry、resume、rollback 或 escalate。

## Debugging Rule

遇到 agent 问题时，先定位风险面，再选择改法：

| Symptom | First place to inspect |
| --- | --- |
| 回答泛泛 | goal contract / instructions / context |
| 工具调用错 | tool description / parameters / routing |
| 多轮混乱 | state strategy / history assembly |
| 长任务崩溃无法恢复 | session log / recovery boundary |
| 擅自执行 | approval policy / tool permission |
| 生成代码能读到密钥 | sandbox / credential boundary |
| 输出难解析 | structured output / schema |
| 不知道为什么错 | trace / logs |
| 修好后又回退 | eval coverage |
| 反复“不够好” | outcome rubric / grader calibration |
| 下次重复犯错 | memory consolidation / practice retro |
