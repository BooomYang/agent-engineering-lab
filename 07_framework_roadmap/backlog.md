# Backlog

## P0

- [x] Trace-to-eval 模板：从一次失败 trace 提取 eval case。
- [ ] Outcome rubric runner：把任务验收标准转成独立 grader、actionable feedback 和 bounded retry policy。
- [ ] Tool spec checklist：评审工具名称、description、schema、副作用、审批和失败语义。
- [ ] Agent design review checklist：评审 agent definition 是否职责过宽、状态混乱或缺少 guardrail。
- [ ] Codex handoff schema checker：检查 handoff 是否包含 trace ids、feedback source、问题分类、目标改动面和验证方式。

## P1

- [ ] Agent production readiness checklist 自动化：读取 scorecard / bundle manifest，检查 trace、eval、guardrail、release gate 是否齐备。
- [ ] Agent bundle manifest：标准化记录 prompt、model/settings、tool schema、knowledge snapshot、guardrails、eval baseline 和 release notes。
- [ ] OpenAI official docs link checker。
- [ ] Multi-agent split decision tree。
- [ ] Multi-agent parallel investigation harness：用于日志、trace、issue、commit 等低耦合资料并行分析。
- [ ] Edge-cloud agent design checklist：面向 iOS/Android 的端侧、边缘云、深度云路由、隐私、权限、缓存和 trace 评审。
- [ ] Human review UI requirement template。
- [ ] Managed-agent runtime interface checklist：检查 brain、session、hands、credential boundary 和 recovery boundary 是否解耦。

## P2

- [ ] Agent engineering starter kit。
- [ ] Eval dataset registry。
- [ ] Trace schema normalization。
- [ ] Agent improvement loop CLI。
- [ ] Memory consolidation review queue：从 traces/practice retros 提取候选记忆，经 review 后进入 skill、template、eval 或项目 memory。

## Parking Lot

- [ ] 和 `loop-engineering-lab` 共用一套复盘到 eval 的流程。
- [ ] 把高频实践沉淀为 Codex skill 或 plugin。
