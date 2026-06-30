# Production Readiness Scorecard

## Metadata

- Agent/workflow:
- Owner:
- Date:
- Status: `draft` / `reviewed` / `blocked` / `approved`
- Related source: https://logic.inc/guides/how-to-build-an-ai-agent
- Related release:

## Scoring Rule

每项按 0/1/2 评分：

- `0`：不存在，或完全依赖人工临场处理。
- `1`：部分存在，但覆盖不完整、不可自动执行或不可稳定复用。
- `2`：已工程化，能在常规开发、发布和运营中稳定执行。

分数只作为 review 入口，不作为通用成熟度标准。低分项必须转成明确 action 或风险接受记录。

## Scorecard

| Area | Score | Evidence | Gap | Next action |
| --- | --- | --- | --- | --- |
| Typed contracts: 输入、输出、工具参数和错误返回有 schema 与 runtime validation。 |  |  |  |  |
| Deterministic tests: schema、业务逻辑、安全拒绝、负约束可自动检查。 |  |  |  |  |
| Probabilistic evals: 有 golden set、失败样本、grader 或人工校准流程。 |  |  |  |  |
| Versioned bundle: prompt、model/settings、tool schema、knowledge、guardrails、eval baseline 可追溯。 |  |  |  |  |
| Observability: 每次 run 可追踪输入、输出、prompt、tool calls、latency、approval 和错误。 |  |  |  |  |
| Model independence: 模型配置集中管理，切换或路由由 eval 和兼容性验证支撑。 |  |  |  |  |
| Release safety: 支持离线回放、shadow run、canary、rollback 或等价的风险控制。 |  |  |  |  |
| Human operation: domain expert 反馈、审批、异常处理和发布记录有明确工作流。 |  |  |  |  |

## Go / No-Go

- Total score:
- Required blockers:
- Accepted risks:
- Decision: `go` / `no-go` / `limited beta`
- Reviewer:

## Validation Plan

- Pre-release eval command or process:
- Shadow/canary plan:
- Rollback trigger:
- Rollback owner:
- Post-release monitoring window:

## Open Questions

- TBD
