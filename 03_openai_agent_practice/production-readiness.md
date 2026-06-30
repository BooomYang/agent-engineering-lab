# Production Readiness

## Official Baseline

上线相关先看：

- Production best practices: https://developers.openai.com/api/docs/guides/production-best-practices
- API deployment checklist: https://developers.openai.com/api/docs/guides/deployment-checklist
- Safety best practices: https://developers.openai.com/api/docs/guides/safety-best-practices

## Readiness Checklist

### Access And Secrets

- [ ] API key 不在代码、日志、截图或仓库中暴露。
- [ ] 开发、测试、生产环境隔离。
- [ ] 生产 key 权限最小化。
- [ ] 密钥轮换和泄漏响应路径明确。

### Rate Limit And Cost

- [ ] 已知模型、token、工具调用、存储、检索等成本来源。
- [ ] 有预算阈值和告警。
- [ ] 有 rate limit、队列或降级策略。
- [ ] 高成本路径有 eval 或人工确认价值。

### Latency And UX

- [ ] 长任务使用 streaming、background mode 或异步状态。
- [ ] UI 能展示“正在思考、正在调用工具、等待审批、失败可重试”等状态。
- [ ] 超时和重试不会重复危险副作用。

### Safety And Permissions

- [ ] 工具按副作用等级分层。
- [ ] 写操作、删除、外部发送、支付、shell、敏感 MCP 有审批策略。
- [ ] 输入、输出、工具调用至少有一个明确 guardrail 设计。
- [ ] 失败时有升级路径，而不是让 agent 猜。

### Observability

- [ ] 每次 run 有 trace 或结构化日志。
- [ ] 能按 user/session/run/tool/error 检索。
- [ ] 能看到 tool parameters、tool results、approval results。
- [ ] 线上失败能转成 eval case。
- [ ] 能定位一次失败对应的 agent bundle 版本、model/settings、tool schema、knowledge snapshot 和 guardrail 配置。

### Evaluation

- [ ] 有最小 golden set。
- [ ] 有真实失败样本。
- [ ] 有安全/审批 eval。
- [ ] 有工具选择和参数 eval。
- [ ] 重要改动前后会跑回归。

### Release And Versioning

- [ ] instructions、tool schema、guardrail、eval set 都有版本。
- [ ] 改动可以回滚。
- [ ] 能对比不同版本 trace 和 eval 结果。
- [ ] 生产变更有发布记录。
- [ ] prompt、model/settings、tools、knowledge、guardrails、eval baseline 作为一个 agent bundle 发布，而不是散落变更。
- [ ] agent logic 和 application code 的生命周期已经区分：行为规则能安全迭代，应用基础设施仍由工程流程控制。
- [ ] 高风险 agent 版本支持 shadow run 或离线回放，先用真实或历史输入比较旧版和新版。
- [ ] 面向用户或外部副作用的 agent 版本支持 canary rollout，而不是一次性切换全量流量。
- [ ] 回滚路径不依赖重新部署完整应用。

### Model Independence

- [ ] 模型名称、provider、temperature、reasoning effort 等配置集中管理。
- [ ] 重要任务先有高能力模型行为基线，再用 eval 比较更快或更低成本模型。
- [ ] 如果使用模型路由，路由条件、fallback 和回归阈值可观测、可测试。
- [ ] 不把“可切换模型”写成承诺，除非 tool calling、structured output、context window 和安全策略都经过兼容性验证。

## Go/No-Go Rule

如果 agent 能触发外部副作用，但没有 trace、审批策略和最小 eval，不应该进入生产。

如果 agent 行为无法追溯到一个明确版本，也不应该进入生产；否则事故发生时很难判断到底是 prompt、model、tool、knowledge、guardrail 还是发布配置变了。
