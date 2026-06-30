# Edge-cloud Agent Architecture

## Evidence Status

本文来自对外部文章《2026 微信 AI 搜索 Agent 全栈实战》的结构化吸收，当前状态是 `external / hypothesis`。

来源链接：https://blog.csdn.net/weixin_42376192/article/details/160007309

其中关于微信、腾讯云、QClaw、ClawBot、规模数字、价格和发布时间的具体说法尚未用官方来源验证。这里沉淀的是可迁移的工程模型，不把文章中的产品细节视为事实基线。

## Core Idea

移动端 agent 不应该默认等于“手机 UI + 云端 agent”。更稳妥的模型是把一个 agent system 拆成多个 runtime tier：

```text
User intent
  -> mobile shell
  -> local intent / privacy / device capability router
  -> local agent runtime or edge/cloud runtime
  -> local tool / remote tool / retrieval / workflow
  -> streamed result
  -> local cache + trace + feedback
```

端侧、边缘云和深度云的差异主要是延迟、隐私、工具权限、算力、成本和可观测性，不天然等同于不同 agent 角色。

## Tier Responsibilities

| Tier | 优先负责 | 不应默认负责 |
| --- | --- | --- |
| 端侧 | 意图预处理、隐私分类、本地缓存、本地检索、端侧工具调用、离线兜底、流式渲染、用户反馈采集 | 大规模知识检索、跨用户聚合、长链路复杂推理、统一策略判断 |
| 边缘云 | 低延迟事实问答、轻量检索、会话续接、短链路工具聚合、流式中转 | 高成本深度推理、敏感私有数据处理、复杂多 agent 编排 |
| 深度云 | 复杂推理、多步计划、大规模检索、多 agent 编排、统一 eval 和 trace 分析 | 需要毫秒响应的交互、必须留在设备内的私有数据处理 |

## Routing Contract

端云协同的关键不是“能否调用云端”，而是每次请求都有可解释的路由理由。

路由 contract 至少包含：

- `task_complexity`：simple、search、workflow、deep_reasoning。
- `privacy_level`：public、user_private、device_private、regulated。
- `device_profile`：model capability、memory、battery、thermal state、OS version。
- `network_profile`：offline、weak、metered、wifi、latency。
- `latency_budget`：用户可接受等待时间。
- `cost_budget`：单次请求可接受成本。
- `tool_authority`：只读、写入、系统权限、支付、消息发送、外部 API。
- `state_scope`：本轮、会话、本地长期、云端长期、多设备同步。
- `fallback_policy`：端侧失败、云端失败、权限拒绝、超时、低电量时如何降级。

## Design Rules

### 1. Deployment Tier Is Not Agent Role

不要因为系统有端侧和云端，就直接拆成“移动端 agent”和“云端 agent”。先判断职责边界：

- 如果只是同一任务在不同算力层执行，优先用一个 agent definition + tiered runtime。
- 如果端侧和云端拥有不同工具、权限、审批策略或输出契约，才考虑拆成 specialist。
- 如果云端只是被端侧调用完成一个子任务，它更像 tool 或 workflow，不一定需要拥有完整 agent 身份。

### 2. Privacy Is A Routing Input

隐私不是事后脱敏，而是路由层的一级输入。

- 设备通讯录、相册、定位、日历、聊天记录、文件内容应默认进入 `device_private`。
- `device_private` 数据优先端侧处理；必须上云时需要明确用户授权、最小化字段、脱敏和审计。
- 本地向量库也需要加密、删除、备份、迁移和多设备同步策略。

### 3. Local Tool Calls Need Authority Boundaries

端侧工具调用比普通 API 更接近真实用户设备权限。

工具 contract 必须写清楚：

- 调用的是系统能力、App 内能力，还是第三方 SDK。
- 是否会读取隐私数据、写入设备状态、发消息、付款或触发外部副作用。
- 是否需要系统弹窗、App 内审批或二次确认。
- 失败时返回什么，是否允许 agent 自动重试。

### 4. Cache Is Part Of State

本地缓存不只是性能优化，它会影响 agent 的事实来源和行为一致性。

需要明确：

- 缓存对象：answer、retrieval result、tool result、embedding、conversation summary。
- 过期策略：按时间、版本、权限、用户撤回、源内容更新。
- 可见性：只对当前设备、当前用户、当前会话，还是多设备同步。
- trace：最终回答是否使用了缓存，缓存何时生成，来源是什么。

### 5. Streamed Results Need Recovery Semantics

移动端网络和 App 生命周期不稳定，流式响应必须考虑恢复：

- 前台切后台时是否继续生成。
- 网络中断后能否恢复或重试。
- 用户取消时云端任务是否取消。
- 已渲染的部分结果是否进入状态。
- 最终 answer 和中间 deltas 如何在 trace 中对齐。

## Mobile Checklist

用于 iOS/Android agent 设计评审：

- [ ] 定义端侧、边缘云、深度云各自承担的任务类型。
- [ ] 定义路由 contract，并能解释每次请求为什么在某一层执行。
- [ ] 区分 deployment tier、agent role、tool 和 workflow。
- [ ] 对每类本地数据标注隐私等级和上云规则。
- [ ] 对每个端侧工具写明权限、副作用、审批、失败语义和重试策略。
- [ ] 设计本地缓存、向量库和会话状态的过期、删除、迁移和审计策略。
- [ ] 设计弱网、离线、低电量、权限拒绝、后台中断、云端超时的降级路径。
- [ ] 把端侧预处理、路由决策、云端推理、工具调用、流式返回和用户反馈串成一条 trace。
- [ ] 用设备档位、网络条件、任务类型、隐私等级和成本预算构造 eval matrix。

## Evals To Add

端云协同 agent 的 eval 不应只看最终回答，还要覆盖路由质量：

| Eval Dimension | Example |
| --- | --- |
| 路由正确性 | 私有通讯录查询是否留在端侧；复杂规划是否升级到云端 |
| 延迟 | 弱网和低端机上的 P50/P95 首 token 时间和完成时间 |
| 成本 | 相同任务在端侧、边缘云、深度云的成本差异 |
| 隐私 | device_private 数据是否被发送到云端或日志 |
| 权限 | 需要系统权限的工具是否触发明确授权 |
| 恢复 | 流式响应中断、App 切后台、用户取消后的状态一致性 |
| 质量 | 端侧快速回答和云端深度回答在关键任务上的质量差距 |

## Open Questions

- 端侧模型、规则路由和传统 ML 分类器各自适合承担哪些路由任务？
- iOS/Android 后台限制会如何影响长任务 agent 和流式恢复？
- 多设备同步时，本地 memory 和云端 state 谁是事实源？
- 端侧 trace 如何脱敏后进入云端 observability 系统？
- 端侧 eval 如何覆盖设备碎片化、系统版本和权限差异？
