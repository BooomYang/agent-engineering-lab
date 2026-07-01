# Agent系统进化论



距离上一篇 [《2025年必看系列：agent开发沉思录，一文理清agent的核心内容》](https://km.woa.com/articles/show/640057)文章分享已经过去了小半年，agent领域的热度不减反增。26年开年的openclaw让非研发同学也体验到了agent的便利，3月至今也见证了半导体的腾飞。 在上周，我们分享了[《Agent系统进化论：当Agent自己学会成长，会发生什么？——自进化的Agent调研》](https://km.woa.com/group/52992/articles/show/631727) 以及Dola自己的agent构建内容[《Agent系统进化论：解密Dola如何构建高准确率、可扩展的自主agent系统》](https://km.woa.com/articles/show/661973)。

在这一篇文章中，笔者把调研过的agent内容汇总到这个系列中，希望帮助更多人了解 Agent 的核心内容。整体是从源码拆解顶级 Coding Agent 的内核，希望它山之石可以攻玉。最后会手把手带你从零搭一个真正能跑的 Agent。

> 💡 **系列定位**：这一篇从顶级 Coding Agent 的源码中提炼设计哲学，再手把手带你从零搭建。

本文虽然主要聚焦于业内顶级 Coding Agent 的源码设计与实现思路，但我们在调研和拆解这些 Agent 的过程中，也不断反哺到 Dola 自身的能力建设与工程实践中。Dola上线一年多以来受到了大家的喜爱，我们最近正在开展一系列的技术分享，如果对技术分享感兴趣的小伙伴，欢迎大家扫码/点击进入交流群！👇🏻

**扫码/点击进入交流群** [点击进群](https://nops.woa.com/pigeon/v1/tools/add_chat?chatId=ww129716153124776)

![](https://km.woa.com/asset/0001000226060002cbd5c136f74ac301?height=268&width=268&imageMogr2/thumbnail/1540x%3E/ignore-error/1)

**点击即刻体验 Dola 👇🏻👇🏼👇🏾**

我们的新一代数据分析agent——[/Dola/](https://dola.woa.com/home)自从推出以来，已经成为鹅厂不少小伙伴的独家数据小黑工了。 如果你是第一次使用Dola的新手，可以试试这些案例👉🏻：股票预测📈（[/帮我预测一下腾讯股价的趋势/](https://dola.woa.com/ai-chat?repo=market-51b9fa0c-d413-46ef-8aef-7ae82667c5cd&model=13&question=%E8%85%BE%E8%AE%AF%E8%82%A1%E4%BB%B7%E8%B6%8B%E5%8A%BF%E9%A2%84%E6%B5%8B&autoSend=true)）、全球大模型调用量分析（[/全球大模型调用排行分析报告/](https://dola.woa.com/share?conversation=3b3a6d2a-0922-4760-961f-2399f7787a17)）、房价预测🏘️（[/深圳房价预测/](https://dola.woa.com/ai-chat?repo=market-b09d21b6-204a-4bb0-be7b-810077919dbb&model=13&question=%E6%B7%B1%E5%9C%B3%E6%9C%AA%E6%9D%A53%E5%B9%B4%E6%88%BF%E4%BB%B7%E8%B6%8B%E5%8A%BF%E9%A2%84%E6%B5%8B&autoSend=true)）； Dola可视化案例：[/Tesla 估值分析报告📈/](https://dola.woa.com/dashboard?spaceId=Dola_game&dashboardId=dash_bAP8DA)、[/全球大模型市场占有率/](https://dola.woa.com/dashboard?spaceId=Dola_game&dashboardId=dash_VBa3JA)、[/腾讯股票未来一月走势预测/](https://dola.woa.com/dashboard?spaceId=Dola_game&dashboardId=dash_ZAoo5A)。 Dola 使用说明见：[附录：Dola使用说明书](https://km.woa.com/articles/show/635996)

* * *

## [](#%E5%BC%80%E7%AF%87%EF%BC%9A%E4%BB%8E%E6%98%AF%E4%BB%80%E4%B9%88%E5%88%B0%E6%80%8E%E4%B9%88%E5%81%9A%E5%88%B0%E7%9A%84)开篇：从"是什么"到"怎么做到的"

2025-2026 年，Coding Agent 全面爆发。Claude Code、OpenAI Codex、Cursor、Windsurf、Cline、Hermes……这些产品走的是不同的设计路线，但在"让 AI 自主完成编程任务"这件事上都做得相当好。

作为 Dola（鹅厂可信数据分析 Agent）核心开发者，过去一年我反复研究了多个顶级开源 Coding Agent 的源码。本文是我的拆解笔记，外加一些实战对照和教学实践。它不光讲"它们为什么强"（设计哲学），还会带你从零搭建一个具备核心能力的 Agent，最后从非技术视角聊聊个人怎么搭建自己的专属 agent。

**为什么要拆 Coding Agent？**

+ 研发同学都在用，甚至不少非技术同学也在用。

+ 它是当前**最成熟**的 Agent 落地场景——有明确的输入输出、可衡量的成功标准、丰富的工具生态

+ 它的设计模式**可迁移**——上下文管理、工具调度、错误恢复等问题，是所有 Agent 的共性挑战

+ 它的代码是**开源的**——可以直接看到生产级实现的每一个细节

* * *

## [](#%E5%89%8D%E7%BD%AE%E7%A7%91%E6%99%AE%EF%BC%9A%E8%AF%BB%E6%87%82%E6%9C%AC%E6%96%87%E6%89%80%E9%9C%80%E7%9A%84%E6%9C%80%E5%B0%8F%E7%9F%A5%E8%AF%86)前置科普：读懂本文所需的最小知识

> 📖 **已读过上一篇的同学可直接跳到 [第一节](#%E4%B8%80coding-agent-%E5%85%A8%E6%99%AF%E5%9B%BE%E4%B8%BA%E4%BB%80%E4%B9%88%E5%80%BC%E5%BE%97%E6%8B%86)**。本节是为零基础读者准备的速查手册，5 分钟内建立必要的心智模型。

### [](#01-agent-%E5%92%8C-chat-%E6%9C%89%E4%BB%80%E4%B9%88%E6%9C%AC%E8%B4%A8%E5%8C%BA%E5%88%AB%EF%BC%9F)0.1 Agent 和 Chat 有什么本质区别？

大多数人接触 AI 的方式是"一问一答"——你发消息，AI 回一条，结束。这就是 **Chat（对话模式）**。

**Agent（智能体）** 的核心区别是：**它能自主循环，直到完成目标**。你交给它一个任务，它会自己规划步骤、调用工具、观察结果，再决定下一步——整个过程不需要你每步确认。

![](https://km.woa.com/asset/00010002260600cc2547b7f2e74fad01?height=1024&width=1536&imageMogr2/thumbnail/1540x%3E/ignore-error/1)

一个任务

思考：下一步做什么？

行动：调用工具

观察：工具返回结果

目标完成？

否，继续

是

👤 你

🤖 Agent

📋 Plan

🔧 Tools

👁️ Observe

✅ Done?

📤 输出结果给你

这个 **Think → Act → Observe** 的循环，就是本文所有源码分析的底层骨架，学术上叫 **ReAct（Reasoning + Acting）**。

### [](#02-%E4%BB%80%E4%B9%88%E6%98%AF%E5%B7%A5%E5%85%B7%E8%B0%83%E7%94%A8%EF%BC%9F)0.2 什么是"工具调用"？

Agent 之所以能完成真实任务，是因为 LLM 可以发出"指令"，让外部程序执行操作并把结果返回给它——这就是**工具调用（Tool Call / Function Call）**。

🔧 Tool（程序）🤖 LLM🔧 Tool（程序）🤖 LLM观察结果，决定下一步{"name": "read\_file", "args": {"path": "main.py"}}{"content": "def main(): ..."}{"name": "bash", "args": {"cmd": "python main.py"}}{"stdout": "Hello World", "exit\_code": 0}

常见工具类型：文件读写、终端命令、网络搜索、数据库查询……工具越丰富，Agent 能完成的任务边界就越大。

### [](#03-%E4%BB%80%E4%B9%88%E6%98%AF%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AA%97%E5%8F%A3%E5%92%8C-token%EF%BC%9F)0.3 什么是"上下文窗口"和 Token？

LLM 每次推理都有一个**输入长度限制**，超出就会报错或遗忘早期内容。这个限制用 **Token** 衡量——大致理解为"词片段"，中文约 1 个汉字 ≈ 1.5 个 Token，英文约 4 个字母 ≈ 1 个 Token。

| 概念         | 通俗理解                                 |
| ---------- | ------------------------------------ |
| Token      | LLM 处理文本的最小单位，约等于"字"或"词片段"           |
| 上下文窗口      | LLM 单次能"看到"的最大 Token 数，Claude 约 20 万 |
| 上下文 = 消息历史 | Agent 每轮都把完整对话历史塞给模型，窗口会越来越满         |

这就是为什么上下文管理是 Agent 最核心的工程挑战。本文第二节会拆解 Claude Code 的五级压缩策略。

### [](#04-%E4%BB%80%E4%B9%88%E6%98%AF-kv-cache%EF%BC%9F)0.4 什么是 KV-Cache？

当你每轮都把相同的前缀（System Prompt + 工具列表 + 历史消息）发给 API 时，云端服务器可以**复用上次计算的中间结果**，不用从头重新计算——这就是 **KV-Cache（键值缓存）**。

对 Agent 的意义：**Cache 命中 → 成本降低 90%，延迟降低 80%**。Claude Code 几乎所有设计决策都在保护这个 Cache，后文会详细分析。

### [](#05-%E6%9C%AC%E6%96%87%E9%98%85%E8%AF%BB%E5%BB%BA%E8%AE%AE)0.5 本文阅读建议

| 你的身份                | 建议阅读路径                                   |
| ------------------- | ---------------------------------------- |
| 🙋 产品 / 运营同学        | 开篇 → 0.x 科普 → 第一节全景图 → 第三节（手把手搭建，看非代码部分） |
| 👩‍💻 研发（刚接触 Agent） | 全文顺序阅读，遇到不懂的术语回查本节                       |
| 🏗️ 研发（有 Agent 经验）  | 可直接跳到第二节源码拆解，重点看 KV-Cache 感知和上下文压缩部分     |

* * *

## [](#%E4%B8%80%E3%80%81coding-agent-%E5%85%A8%E6%99%AF%E5%9B%BE%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E5%80%BC%E5%BE%97%E6%8B%86%EF%BC%9F)一、Coding Agent 全景图：为什么值得拆？

### [](#%E6%A0%B8%E5%BF%83%E9%97%AE%E9%A2%98%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E6%95%88%E6%9E%9C%E5%B7%AE%E5%BC%82%E5%A4%A7%EF%BC%9F)核心问题：为什么效果差异大？

所有 Coding Agent 本质上都在做同一件事：**while(true) { 思考 → 行动 → 观察 }**。但为什么有的产品一句话就能完成复杂重构，有的却在简单任务上反复犯错？

答案不在模型能力（大家用的是同一批模型），而在**架构设计**——尤其是以下四个维度：

| 维度            | 决定了什么         |
| ------------- | ------------- |
| Agent Loop 设计 | 模型如何持续思考和行动   |
| 上下文管理         | 模型能"看到"多少有效信息 |
| 工具调度          | 模型能"做"什么、做得多快 |
| 错误恢复          | 出错后能否自愈而非崩溃   |

接下来，我们从五个代表性项目的源码中，逐一拆解这些维度的**最佳实践**。

* * *

## [](#%E4%BA%8C%E3%80%81%E6%A0%B8%E5%BF%83%E5%AF%B9%E6%AF%94%E4%B8%8E%E6%BA%90%E7%A0%81%E6%8B%86%E8%A7%A3)二、核心对比与源码拆解

### [](#21-claude-code-%E2%80%94-kv-cache-%E6%84%9F%E7%9F%A5%E6%98%AF%E7%AC%AC%E4%B8%80%E5%85%AC%E6%B0%91)2.1 Claude Code — "KV-Cache 感知是第一公民"

> 🏷️ 语言：TypeScript/Bun | 架构：单循环流式 Agent | 核心哲学：极简但强大

Claude Code 的架构可以用一句话概括：所有设计决策都围绕"如何最大化 Prompt Cache 命中"展开。这不只是性能优化的问题。当你的 Agent 每一轮对话都要传入完整上下文时，Cache 命中率会直接决定成本和延迟。

#### [](#agent-loop%EF%BC%9Aasyncgenerator-%E7%8A%B6%E6%80%81%E6%9C%BA)Agent Loop：AsyncGenerator + 状态机

Claude Code 的核心循环位于 `src/query.ts`，采用 **AsyncGenerator + while(true)** 模式：

```js
// src/query.ts - 核心循环骨架
async function* queryLoop(params: QueryParams): AsyncGenerator<StreamEvent, Terminal> {
  // 可变的跨迭代状态
  let state: State = {
    messages: params.messages,
    autoCompactTracking: undefined,
    maxOutputTokensRecoveryCount: 0,
    hasAttemptedReactiveCompact: false,
    turnCount: 1,
    transition: undefined,  // 转移原因：compact_retry / escalate / continuation
  }

  while (true) {
    const { messages, turnCount } = state
    yield { type: 'stream_request_start' }

    // ====== 分级压缩管线（从轻到重） ======
    let messagesForQuery = [...getMessagesAfterCompactBoundary(messages)]
    messagesForQuery = snipCompact(messagesForQuery)         // Level 1: 裁剪旧消息
    messagesForQuery = microCompact(messagesForQuery)        // Level 2: 压缩单条 tool_result
    // Level 3 & 4: Context Collapse / AutoCompact 按条件触发

    // ====== LLM 调用 + 流式工具执行 ======
    const streamingToolExecutor = new StreamingToolExecutor(tools, canUseTool)
    for await (const message of callModel({ messages: messagesForQuery, ... })) {
      yield message  // 流式透传给消费者
      if (message.type === 'assistant') {
        for (const toolBlock of message.content.filter(c => c.type === 'tool_use')) {
          streamingToolExecutor.addTool(toolBlock)  // 边收边执行，不等响应结束！
        }
      }
    }

    // ====== 工具结果收集 + 错误恢复 ======
    const toolResults = await streamingToolExecutor.getResults()
    // ... 错误检测、withhold、重试逻辑 ...

    if (!needsFollowUp) break  // 没有工具调用 → 结束
  }
}
```

**关键设计点**：

+ **AsyncGenerator 而非 Promise**：允许调用方逐条消费消息，实现真正的流式体验

+ **状态机驱动转移**：`transition` 字段记录为什么要"再来一轮"——是压缩后重试？是 token 超限升级？还是续写？

+ **边收边执行**：不等 API 响应结束就开始执行工具，极大降低体感延迟

#### [](#%E5%B7%A5%E5%85%B7%E7%B3%BB%E7%BB%9F%EF%BC%9Astreamingtoolexecutor-%E6%99%BA%E8%83%BD%E5%B9%B6%E5%8F%91)工具系统：StreamingToolExecutor + 智能并发

```js
// src/services/tools/StreamingToolExecutor.ts
class StreamingToolExecutor {
  private pendingTools: Map<string, { tool: ToolUseBlock; promise: Promise<ToolResult> }> = new Map()
  private concurrentBatch: ToolUseBlock[] = []
  private serialQueue: ToolUseBlock[] = []

  addTool(toolBlock: ToolUseBlock): void {
    const toolDef = this.tools.find(t => t.name === toolBlock.name)
    if (toolDef?.isConcurrencySafe) {
      this.concurrentBatch.push(toolBlock)  // 并发安全 → 立即执行
      this.startExecution(toolBlock)
    } else {
      this.serialQueue.push(toolBlock)       // 非并发安全 → 排队等待
    }
  }

  async getResults(): Promise<ToolResult[]> {
    // 先等并发批完成，再串行执行队列
    await Promise.all(this.concurrentBatch.map(t => this.pendingTools.get(t.id)!.promise))
    for (const tool of this.serialQueue) {
      await this.executeSerially(tool)
    }
    return this.collectResults()
  }
}
```

每个 Tool 定义通过 `isConcurrencySafe` 声明自己是否可并发：

+ `ReadFile`、`SearchContent`、`ListDir` → 并发安全，最多 10 个并行

+ `WriteFile`、`ExecuteCommand` → 非并发安全，严格串行

#### [](#%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AE%A1%E7%90%86%EF%BC%9A%E4%BA%94%E7%BA%A7%E9%80%92%E8%BF%9B%E5%8E%8B%E7%BC%A9)上下文管理：五级递进压缩

这是 Claude Code 里我最欣赏的一处设计：不是一上来就全量压缩，而是从轻到重逐级尝试。

![](https://km.woa.com/asset/000100022606004e63f58c582b438e01?height=1536&width=1024&imageMogr2/thumbnail/1540x%3E/ignore-error/1)

| 级别  | 名称                 | 触发条件    | 代价        | 策略                                |
| --- | ------------------ | ------- | --------- | --------------------------------- |
| L0  | Tool Output Budget | 每轮都执行   | 零 LLM 开销  | 工具输出超出预算即持久化到磁盘，上下文只保留预览          |
| L1  | Snip Compact       | 每轮都执行   | 零 LLM 开销  | 裁剪最早的消息，只保留最近 N 条                 |
| L2  | MicroCompact       | 每轮都执行   | 零 LLM 开销  | 按时间衰减压缩单条 tool\_result（越老压缩越狠）    |
| L3  | Context Collapse   | ~85% 窗口 | 一次 LLM 调用 | 折叠式——生成结构化摘要但保留关键细节               |
| L4  | AutoCompact        | ~90% 窗口 | 较重 LLM 调用 | 全量总结式，6 阶段流水线（提取→分类→优先级→摘要→验证→替换） |

**L0 补充：工具输出预算（Tool Output Budget）**

在进入任何压缩策略之前，Claude Code 首先对 **工具调用的原始输出** 设置了严格的字符预算，超出阈值的内容会被持久化到本地磁盘，上下文中仅保留一段 **2000 字节的预览**：

```js
// src/constants/toolLimits.ts — 核心预算常量
const DEFAULT_MAX_RESULT_SIZE_CHARS = 50_000       // 单个工具输出上限：5 万字符
const MAX_TOOL_RESULTS_PER_MESSAGE_CHARS = 200_000 // 单条 user message 所有工具输出总和上限：20 万字符
const PREVIEW_SIZE_BYTES = 2_000                   // 持久化后保留的预览大小

// src/utils/toolResultStorage.ts — 超出阈值时持久化到磁盘
async function maybePersistLargeToolResult(toolResultBlock, toolName) {
  const size = contentSize(content)
  const threshold = getPersistenceThreshold(toolName, tool.maxResultSizeChars)

  if (size <= threshold) return toolResultBlock  // 未超出，直接返回

  // 超出：写入 {sessionDir}/tool-results/{tool_use_id}.txt
  const result = await persistToolResult(content, toolResultBlock.tool_use_id)
  // 上下文中替换为预览消息：
  // "<persisted-output>\nOutput too large (XXX chars). Full output saved to: {path}\nPreview:\n..."
  return { ...toolResultBlock, content: buildLargeToolResultMessage(result) }
}
```

两层预算的执行逻辑：

+ **per-tool 预算**：每个工具自己声明 `maxResultSizeChars`，实际阈值取 `Math.min(工具声明值, 50_000)`。`FileReadTool` 声明为 `Infinity`（永不持久化，因为持久化会触发"读文件→持久化→再读持久化文件"的死循环）；`BashTool` `GrepTool` 上限分别为 30k 20k 字符

+ **per-message 聚合预算**：一次并行工具调用（同一条 user message）的所有输出字符总和超过 20 万时，从最大的工具结果开始贪心地依次持久化，直到总量满足预算

持久化写入时使用 `flag: 'wx'`（文件已存在则跳过），保证 MicroCompact 在重放历史消息时不会重复写入同一文件。

```js
// 自动触发逻辑（简化）
if (tokenUsage > 0.97) {
  // 阻塞！必须压缩才能继续
  await forceAutoCompact(messages)
} else if (tokenUsage > 0.90) {
  // 自动触发，但有熔断器保护（连续失败3次则停止）
  if (autoCompactCircuitBreaker.isOpen()) return
  await autoCompact(messages)
}
```

#### [](#kv-cache-%E6%84%9F%E7%9F%A5%E8%AE%BE%E8%AE%A1%EF%BC%88%E6%A0%B8%E5%BF%83%E4%B8%AD%E7%9A%84%E6%A0%B8%E5%BF%83%EF%BC%89)KV-Cache 感知设计（核心中的核心）

Claude API 支持 Prompt Cache。如果你的请求前缀和上一次相同，Anthropic 会复用已缓存的 KV-Cache，成本降低 90%，延迟降低 80%。Claude Code 的所有设计都在保护这个 Cache：

![](https://km.woa.com/asset/0001000226060040205790de4c465f01?height=1024&width=1536)

```js
// 1. 工具按名称排序 → 工具列表不变 = Cache 命中
const sortedTools = tools.sort((a, b) => a.name.localeCompare(b.name))

// 2. cache_control 标记只放在最后一条消息 → 保护前面所有内容的 Cache
messages[messages.length - 1].cache_control = { type: 'ephemeral' }

// 3. Compact 方向选择 'from'（从前往后保留）→ 保护 prefix
compact({ direction: 'from', messages })

// 4. CacheSafeParams 共享 → Fork Agent 复用父请求的 system prompt + tools + model
const subAgent = fork({ cacheSafeParams: parent.cacheSafeParams })

// 5. 追踪每次 Cache 断裂原因
promptCacheBreakDetection.record({ reason: 'tool_list_changed', impact: estimatedCost })
```

> 📌 **实践建议**：
>
> + System Prompt 和工具列表**一旦确定就别动**——每次变更都可能导致 Cache 失效
>
> + 如果你用 Anthropic API，把 `cache_control` 放在对话最后一条消息上
>
> + 新增工具时按名称字典序插入，而非追加到末尾

* * *

### [](#22-openai-codex-%E2%80%94-%E5%AE%89%E5%85%A8%E5%8D%B3%E6%9E%B6%E6%9E%84)2.2 OpenAI Codex — "安全即架构"

> 🏷️ 语言：Rust(核心) + TypeScript/Python(SDK) | 架构：沙箱隔离 + 多Agent协作 | 核心哲学：安全是第一架构约束

Codex 与 Claude Code 最大的区别在于：它不信任 Agent 执行的任何代码。所有代码执行都放在操作系统级沙箱里。这里的安全不是事后补上的，而是从架构设计之初就定下的核心约束。

#### [](#%E6%A0%B8%E5%BF%83%E5%BE%AA%E7%8E%AF%EF%BC%9Arust-%E5%AE%9E%E7%8E%B0%E7%9A%84-turn-loop)核心循环：Rust 实现的 Turn Loop

```js
// codex-rs/core/src/session/turn.rs - 核心 turn 循环
pub(crate) async fn run_turn(
    sess: Arc<Session>,
    turn_context: Arc<TurnContext>,
) -> Option<String> {
    loop {
        // 1. 收集待处理输入
        let pending_input = sess.input_queue.get_pending_input(&sess.active_turn).await;

        // 2. 构建模型请求
        let sampling_request_input: Vec<ResponseItem> = sess.clone_history().await
            .for_prompt(&turn_context.model_info.input_modalities);

        // 3. 执行采样请求
        match run_sampling_request(sess.clone(), turn_context.clone(), sampling_request_input).await {
            Ok(result) => {
                let SamplingRequestResult { needs_follow_up, last_agent_message } = result;

                if !needs_follow_up {
                    break;  // 模型没有请求工具调用 → Turn 结束
                }

                // Token 限制触发自动压缩
                if token_limit_reached && needs_follow_up {
                    run_auto_compact(/* ... */).await;
                }
                continue;  // function_call 执行完毕，继续下一轮采样
            }
            Err(CodexErr::TurnAborted) => break,
            Err(e) => { sess.send_event(EventMsg::Error(e)).await; break; }
        }
    }
    last_agent_message
}
```

#### [](#%E6%B2%99%E7%AE%B1%E6%9E%B6%E6%9E%84%EF%BC%9A%E4%B8%89%E5%B9%B3%E5%8F%B0-os-%E7%BA%A7%E9%9A%94%E7%A6%BB)沙箱架构：三平台 OS 级隔离

![](https://km.woa.com/asset/00010002260600cee2b1d48b79430d01?height=1024&width=1536)

```js
// codex-rs/core/src/sandbox/linux.rs - Linux 沙箱创建（Bubblewrap）
fn create_sandbox_command(config: &SandboxConfig) -> Command {
    let mut cmd = Command::new("bwrap");
    cmd.args([
        "--unshare-user",      // 用户命名空间隔离
        "--unshare-pid",       // PID 命名空间隔离
        "--unshare-net",       // 网络命名空间隔离（完全断网）
        "--die-with-parent",   // 父进程退出时自动杀死
    ]);

    // 文件系统策略
    match config.policy {
        ReadOnly => {
            cmd.args(["--ro-bind", &workspace, &workspace]);  // 只读绑定
        }
        WorkspaceWrite => {
            cmd.args(["--bind", &workspace, &workspace]);      // 可写
            cmd.args(["--ro-bind", &git_dir, &git_dir]);       // .git 强制只读！
        }
        FullAccess => { /* 仅容器环境允许 */ }
    }

    // Seccomp 系统调用过滤
    cmd.args(["--seccomp", &seccomp_filter_fd]);
    cmd
}
```

**三级文件系统策略**遵循的是"最小权限原则"：

+ **read-only**：Agent 只能看代码，不能改（用于代码审查场景）

+ **workspace-write**：可以改代码但 `.git` 强制只读（防止 Agent 篡改版本历史）

+ **danger-full-access**：完全不限制（仅在 Docker 容器中允许）

#### [](#%E5%B7%A5%E5%85%B7%E7%BC%96%E6%8E%92%EF%BC%9A%E4%BA%94%E5%B1%82%E5%88%86%E5%B1%82%E6%9E%B6%E6%9E%84)工具编排：五层分层架构

```js
ToolSpec（模型可见定义，JSON Schema）
    ↓
ToolRouter（路由分发，匹配工具名→实现）
    ↓
ToolRegistry（运行时注册表，支持动态注册/注销）
    ↓
ToolOrchestrator（审批 + 沙箱选择 + 重试策略）
    ↓
ToolRuntime（具体执行：Shell / FileSystem / Network）
```

其中 **ToolOrchestrator** 最值得学习——它引入了"审批缓存"（ApprovalStore）：

```js
// 审批缓存逻辑（简化）
async fn execute_with_approval(tool_call: &ToolCall) -> ToolResult {
    // 1. 检查审批缓存：相同类型操作是否已被批准？
    if approval_store.is_approved(&tool_call.approval_key()) {
        return execute_in_sandbox(tool_call).await;  // 直接执行
    }

    // 2. 请求用户审批
    let decision = request_approval(tool_call).await;
    match decision {
        Approve(scope) => {
            approval_store.cache(tool_call.approval_key(), scope);  // 缓存审批
            execute_in_sandbox(tool_call).await
        }
        Deny => ToolResult::Error("User denied".into()),
    }
}
```

用户批准一次"允许写入 src/ 目录"后，后续对 `src/` 的写入就不必再确认，安全和流畅都顾上了。

#### [](#%E5%A4%9A-agent-%E5%8D%8F%E4%BD%9C%E5%8E%9F%E8%AF%AD)多 Agent 协作原语

Codex 内置了 Agent 间通信的原语：

```js
// 系统提示中鼓励并行
"Prefer multiple sub-agents to parallelize independent tasks."

// 5 个核心原语
spawn_agent({ name: "test-runner", task: "Run all unit tests" })
wait_agent({ name: "test-runner" })
send_message({ to: "test-runner", content: "Focus on auth module" })
close_agent({ name: "test-runner" })
list_agents()
```

#### [](#%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AE%A1%E7%90%86%EF%BC%9Acontextmanager-%E4%B8%89%E9%98%B6%E6%AE%B5%E8%87%AA%E5%8A%A8%E5%8E%8B%E7%BC%A9)上下文管理：ContextManager + 三阶段自动压缩

与 Claude Code 的 Compact 不同，Codex 的上下文管理是一套 **Rust 实现的精密 Token 预算系统**，核心围绕 `ContextManager` + `AutoCompactWindow` + 三阶段压缩触发构建。

**核心数据结构 — ContextManager**：

```js
// codex-rs/core/src/context_manager/history.rs - 对话历史管理器
#[derive(Debug, Clone, Default)]
pub(crate) struct ContextManager {
    /// 有序对话历史（从旧到新）
    items: Vec<ResponseItem>,
    /// 每次历史被重写（压缩/回滚）时递增，用于版本追踪
    history_version: u64,
    /// 实时 Token 使用统计（来自服务端响应）
    token_info: Option<TokenUsageInfo>,
    /// 上下文差分基线：用于下一轮只发送"变化量"而非完整上下文
    reference_context_item: Option<TurnContextItem>,
}
```

**关键设计：差分注入（Incremental Context）**——不是每轮都发全量 System Prompt：

```js
// codex-rs/core/src/session/mod.rs - 上下文注入的两条路径
pub(crate) async fn record_context_updates_and_set_reference_context_item(&self, turn_context: &TurnContext) {
    let reference_context_item = state.reference_context_item();
    let should_inject_full_context = reference_context_item.is_none();

    let context_items = if should_inject_full_context {
        // 路径 A：首次 / 压缩后 → 完整注入初始上下文
        self.build_initial_context(turn_context).await
    } else {
        // 路径 B：稳态路径 → 仅追加上下文差分，最小化 Token 开销
        self.build_settings_update_items(reference_context_item.as_ref(), turn_context).await
    };

    // 更新差分基线，下一轮继续增量更新
    state.set_reference_context_item(Some(turn_context.to_turn_context_item()));
}
```

**三阶段自动压缩触发**：

```js
// codex-rs/core/src/session/turn.rs - Token 预算判断与压缩触发
// 阶段 1：Pre-Turn（采样前检查）
async fn run_pre_sampling_compact(sess, turn_context, client_session) {
    let token_status = auto_compact_token_status(sess, turn_context).await;
    if token_status.token_limit_reached {
        run_auto_compact(sess, turn_context, client_session,
            InitialContextInjection::DoNotInject,   // 压缩后清空基线，下一轮完整注入
            CompactionReason::ContextLimit,
            CompactionPhase::PreTurn,               // 阶段标记
        ).await?;
    }
}

// 阶段 2：Mid-Turn（采样中途触发）
// 在 turn loop 中，每次模型返回后检查
if token_limit_reached && needs_follow_up {
    run_auto_compact(sess, turn_context, client_session,
        InitialContextInjection::BeforeLastUserMessage, // 保留最后用户消息上方的上下文
        CompactionReason::ContextLimit,
        CompactionPhase::MidTurn,
    ).await?;
}

// 阶段 3：Model Downshift（模型降级时压缩）
// 当切换到更小上下文窗口的模型时，预先压缩
if previous_model_limit_reached && old_context_window > new_context_window {
    run_auto_compact(sess, &previous_model_turn_context, client_session,
        InitialContextInjection::DoNotInject,
        CompactionReason::ModelDownshift,
        CompactionPhase::PreTurn,
    ).await?;
}
```

**压缩策略选择 — 三路分发**：

```js
// codex-rs/core/src/session/turn.rs - 根据 Provider 能力选择压缩实现
async fn run_auto_compact(sess, turn_context, ...) {
    if should_use_remote_compact_task(turn_context.provider.info()) {
        if turn_context.features.enabled(Feature::RemoteCompactionV2) {
            // 方案 A：Remote V2 — 服务端流式压缩（最新方案）
            run_inline_remote_auto_compact_task_v2(...).await?;
        } else {
            // 方案 B：Remote V1 — 服务端批量压缩
            run_inline_remote_auto_compact_task(...).await?;
        }
    } else {
        // 方案 C：Local — 本地用 LLM 生成摘要
        run_inline_auto_compact_task(...).await?;
    }
}
```

**压缩执行核心 — "摘要 + 保留最近用户消息"**：

```js
// codex-rs/core/src/compact.rs - 压缩后历史重建
// 1. 让模型对当前历史生成摘要
let summary_suffix = get_last_assistant_message_from_turn(history_items).unwrap_or_default();
let summary_text = format!("{SUMMARY_PREFIX}\n{summary_suffix}");
// SUMMARY_PREFIX = "Another language model started to solve this problem and produced
//                   a summary of its thinking process..."

// 2. 保留最近的用户消息（最多 20000 tokens），从后往前选取
let user_messages = collect_user_messages(history_items);
let new_history = build_compacted_history(Vec::new(), &user_messages, &summary_text);

// 3. 如果是 Mid-Turn 压缩，在最后用户消息上方注入完整初始上下文
if initial_context_injection == BeforeLastUserMessage {
    let initial_context = sess.build_initial_context(turn_context).await;
    new_history = insert_initial_context_before_last_real_user_or_summary(new_history, initial_context);
}

// 4. 替换历史，递增版本号，清空压缩窗口计数
sess.replace_compacted_history(new_history, reference_context_item, compacted_item).await;
```

压缩后的 Prompt 内容如下（`codex-rs/prompts/templates/compact/prompt.md`）：

> You are performing a CONTEXT CHECKPOINT COMPACTION. Create a handoff summary for another LLM that will resume the task. Include: Current progress and key decisions made Important context, constraints, or user preferences What remains to be done (clear next steps) / Any critical data, examples, or references needed to continue.

**工具输出截断 — 防止单条消息爆 Token**：

```js
// codex-rs/core/src/context_manager/history.rs - 记录时自动截断
pub(crate) fn record_items<I>(&mut self, items: I, policy: TruncationPolicy) {
    for item in items {
        if !is_api_message(item_ref) { continue; }
        let processed = self.process_item(item_ref, policy);  // 按 TruncationPolicy 截断
        self.items.push(processed);
    }
}
// TruncationPolicy 通常设为 token 上限，超长的工具输出会被截断保留首尾
```

**历史规范化 — 发送前的三重保障**：

```js
// codex-rs/core/src/context_manager/history.rs - for_prompt 前的规范化
fn normalize_history(&mut self, input_modalities: &[InputModality]) {
    // 不变量 1：每个函数调用都有对应的输出条目（补空输出）
    normalize::ensure_call_outputs_present(&mut self.items);
    // 不变量 2：每个输出都有对应的函数调用（孤立输出被移除）
    normalize::remove_orphan_outputs(&mut self.items);
    // 不变量 3：不支持图片的模型，剥离所有图片内容
    if !input_modalities.contains(&InputModality::Image) {
        strip_images(&mut self.items);
    }
}
```

**AutoCompactWindow — Token 窗口记账**：

```js
                   ┌────── AutoCompactWindow ──────┐
                   │                               │
 [压缩摘要]        │ [prefill]  [新增body tokens]  │
                   │    ↑           ↑              │
                   │ 压缩后首轮   后续轮次累加       │
                   │ 的token数    的新增token       │
                   │                               │
                   │ 触发条件：body > limit         │
                   └───────────────────────────────┘
```

Codex 支持两种 Token 预算范围（`AutoCompactTokenLimitScope`）：

+ **Total**：当总 token 超过 `auto_compact_token_limit` 时触发

+ **BodyAfterPrefix**：只计算压缩后新增的 token（排除 prefill），对长任务更友好

> 📌 **实践建议**：
>
> + 代码执行必须有沙箱——哪怕最简单的 Docker 隔离也好过裸执行
>
> + "审批缓存"模式兼顾安全和体验：首次审批，后续自动放行同类操作
>
> + 如果你的 Agent 任务复杂度高，考虑"spawn 子 Agent 并行"而非"单 Agent 串行"
>
> + 上下文管理的"差分注入"思路值得学习——非首轮对话只发变更部分，大幅节省 Token
>
> + 压缩时保留最近用户消息 + 结构化摘要，比简单的"截断头部"丢失信息更少

* * *

### [](#23-hermes-agent-%E2%80%94-%E8%87%AA%E8%BF%9B%E5%8C%96-prompt-cache-%E7%AC%AC%E4%B8%80%E5%85%AC%E6%B0%91)2.3 Hermes Agent — "自进化 + Prompt Cache 第一公民"

> 🏷️ 语言：Python | 架构：单循环 + 自进化闭环 | 核心哲学：模型无锁定 + 经验沉淀

Hermes 的两大特色：1）完全兼容 OpenAI 协议，一行命令切换任意模型；2）完成任务后**自动沉淀 Skill**，Agent 越用越强。

#### [](#%E6%A0%B8%E5%BF%83%E5%BE%AA%E7%8E%AF%EF%BC%9Aiterationbudget-%E9%A2%84%E7%AE%97%E5%88%B6)核心循环：IterationBudget 预算制

```js
# agent/conversation_loop.py - 主循环
while (api_call_count < agent.max_iterations
       and agent.iteration_budget.remaining > 0) or agent._budget_grace_call:

    # 中断检测
    if agent._interrupt_requested:
        break

    api_call_count += 1

    # 预算消耗（线程安全）
    if agent._budget_grace_call:
        agent._budget_grace_call = False  # Grace call: 预算耗尽后的"最后一次机会"
    elif not agent.iteration_budget.consume():
        break

    # === Token 预检 + 上下文压缩 ===
    estimated_tokens = count_tokens(messages)
    if estimated_tokens > context_window * 0.85:
        messages = await context_compressor.compress(messages)  # 最多重试 3 轮
        compression_attempts += 1
        if compression_attempts > 3:
            # Session 分裂：创建 child session，携带压缩摘要继续
            session = session.split(summary=generate_handoff_summary(messages))

    # === API 调用 ===
    response = await call_model(messages, tools=get_tool_definitions())

    # === 工具执行（最多 8 线程并发） ===
    if response.tool_calls:
        results = await execute_tools_concurrent(response.tool_calls, max_workers=8)
        messages.extend(results)
    else:
        final_response = response.content
        break
```

**IterationBudget** 是一个巧妙的设计：

```js
# agent/iteration_budget.py
class IterationBudget:
    """线程安全的迭代计数器。父 Agent 默认 90 轮，子 Agent 50 轮。"""

    def __init__(self, max_total: int):
        self.max_total = max_total
        self._used = 0
        self._lock = threading.Lock()

    def consume(self) -> bool:
        """尝试消耗一次迭代。返回 True 表示允许。"""
        with self._lock:
            if self._used >= self.max_total:
                return False
            self._used += 1
            return True

    def refund(self) -> None:
        """退还一次迭代（如 execute_code 不消耗预算）。"""
        with self._lock:
            if self._used > 0:
                self._used -= 1
```

`refund()` 方法特别有意思。某些"纯执行"的迭代（比如运行测试脚本）不该消耗思考预算，可以退还。

#### [](#%E5%B7%A5%E5%85%B7%E6%B3%A8%E5%86%8C%EF%BC%9Aast-%E8%87%AA%E5%8A%A8%E5%8F%91%E7%8E%B0)工具注册：AST 自动发现

Hermes 的工具注册是声明式的：新增一个工具只需在 `tools/` 目录下创建文件，不用改动任何其他代码。

```js
# tools/registry.py - AST 自动发现
import ast
from pathlib import Path

def _is_registry_register_call(node: ast.AST) -> bool:
    """检测是否为 registry.register(...) 调用。"""
    if not isinstance(node, ast.Expr) or not isinstance(node.value, ast.Call):
        return False
    func = node.value.func
    return (isinstance(func, ast.Attribute) and func.attr == "register"
            and isinstance(func.value, ast.Name) and func.value.id == "registry")

def discover_builtin_tools() -> list[str]:
    """扫描 tools/ 目录，通过 AST 检测哪些模块注册了工具。"""
    tools_dir = Path(__file__).parent
    discovered = []
    for py_file in tools_dir.glob("*.py"):
        if py_file.name.startswith("_"):
            continue
        try:
            tree = ast.parse(py_file.read_text(), filename=str(py_file))
            if any(_is_registry_register_call(node) for node in ast.iter_child_nodes(tree)):
                discovered.append(py_file.stem)
        except (OSError, SyntaxError):
            continue
    return discovered
```

为什么用 AST 而不是直接 import？因为 import 有副作用：某些工具模块可能依赖 Docker、GPU 等外部资源，而 AST 解析不会触发这些副作用。

#### [](#%E8%87%AA%E8%BF%9B%E5%8C%96-skill-%E9%97%AD%E7%8E%AF)自进化 Skill 闭环

这是 Hermes 最独特的设计：Agent 完成复杂任务后，会自动把经验沉淀为可复用的 Skill。

```js
# skills/creator.py - Skill 创建流程（简化）
async def maybe_create_skill(task_trajectory: list[Message]) -> Optional[Skill]:
    """任务完成后，评估是否值得创建 Skill。"""
    # 1. 判断任务复杂度是否值得沉淀
    if len(task_trajectory) < 5:
        return None  # 太简单的任务不值得

    # 2. 提取任务模式
    pattern = await extract_task_pattern(task_trajectory)

    # 3. 生成 SKILL.md（纯 Markdown 格式）
    skill_content = f"""# {pattern.name}

## 触发条件
{pattern.trigger_description}

## 执行步骤
{pattern.steps_markdown}

## 注意事项
{pattern.caveats}
"""
    # 4. 保存为 Skill 文件
    skill_path = SKILLS_DIR / f"{pattern.name}.md"
    skill_path.write_text(skill_content)

    # 5. 注册到 Skill 索引
    register_skill(skill_path, patch_count=0)
    return Skill(path=skill_path)
```

**自进化闭环**：创建 Skill → 后续任务匹配到 Skill → 执行中发现不完善 → 自动 patch → `patch_count` 递增 → Curator 守护进程定期清理低质量 Skill。

> 📌 **实践建议**：
>
> + "AST 自动发现"是工具扩展的最佳实践：新增工具 = 新增文件，零修改核心代码
>
> + IterationBudget + Grace Call 模式值得借鉴：防止 Agent 无限循环，又给它"最后一次机会"体面收尾
>
> + Skill 沉淀思路：Agent 的经验应该结构化保存，而非消失在日志里

* * *

### [](#24-%E6%89%A9%E5%B1%95%E5%AF%B9%E7%85%A7)2.4 扩展对照

#### [](#openclaw-%E2%80%94-%E5%A4%9A-agent-%E7%BC%96%E6%8E%92%E7%9A%84%E6%8E%A7%E5%88%B6%E5%B9%B3%E9%9D%A2)OpenClaw — 多 Agent 编排的"控制平面"

OpenClaw 的着眼点不在"造一个更聪明的 Agent"，而是造一个管理 Agent 的操作系统：

```js
┌─────────────────────────────────────────────────┐
│                  Gateway（控制平面）               │
│    WebSocket RPC + 22 种消息渠道统一路由           │
├─────────────────────────────────────────────────┤
│  Bindings 路由表                                  │
│  channel=whatsapp + peer=+86xxx → Agent-A        │
│  channel=slack    + guild=xxx   → Agent-B        │
│  channel=telegram + peer=xxx    → Agent-C        │
├─────────────────────────────────────────────────┤
│  Agent-A          Agent-B          Agent-C       │
│  (独立工作区)      (独立工作区)      (独立工作区)  │
│  AGENTS.md        AGENTS.md        AGENTS.md     │
│  SOUL.md          SOUL.md          SOUL.md       │
└─────────────────────────────────────────────────┘
```

**核心设计哲学**：

+ **"Prompt as Code"**：AGENTS.md(行为指令) + SOUL.md(人格) + TOOLS.md(工具说明) + BOOT.md(启动逻辑) + HEARTBEAT.md(心跳) → Agent 行为完全版本化管理

+ **扁平协作模型**：拒绝 Manager-of-Managers 层级框架，用 session-based 的扁平通信

+ **Node 模式**：设备（macOS/iOS/Android）作为远程节点，暴露 camera/screen/location 等能力

#### [](#google-adk-python-%E2%80%94-%E5%A3%B0%E6%98%8E%E5%BC%8F-agent-%E6%A1%86%E6%9E%B6%E7%9A%84%E5%85%B8%E8%8C%83)Google ADK-Python — 声明式 Agent 框架的典范

ADK 的核心设计是"函数即工具、代码即定义"：用 Python 代码而不是配置文件来定义所有东西。

```js
# 定义工具：就是一个普通函数！
def get_weather(city: str) -> str:
    """获取指定城市的天气信息。"""
    return f"{city} 今天 25°C 晴"

# 定义 Agent：声明式组合
weather_agent = LlmAgent(
    name="weather_assistant",
    model="gemini-2.0-flash",
    tools=[get_weather],  # 直接传函数！自动生成 JSON Schema
    sub_agents=[detail_agent, summary_agent],
)
```

框架内部通过 `inspect.signature` 自动从函数签名提取参数类型，生成 OpenAPI 格式的 `FunctionDeclaration`，几乎不用写胶水代码：

```js
# src/google/adk/tools/function_tool.py
class FunctionTool(BaseTool):
    def __init__(self, func: Callable[..., Any]):
        self.func = func
        # 自动从签名生成 Schema
        self._declaration = build_function_declaration(func=func)

    async def run_async(self, *, args: dict, tool_context: ToolContext) -> Any:
        # 自动注入 tool_context 参数
        signature = inspect.signature(self.func)
        if 'tool_context' in signature.parameters:
            args['tool_context'] = tool_context
        return await self._invoke_callable(self.func, args)
```

**Workflow 图引擎**（2.0 新增）支持声明式 DAG：

```js
from google.adk.workflow import Workflow, node, START, END

@node
async def validate_input(ctx): ...

@node
async def process_data(ctx): ...

@node
async def generate_report(ctx): ...

# 声明式定义 DAG
pipeline = Workflow(
    edges=[
        (START, validate_input, process_data),  # 链式语法
        (process_data, generate_report, END),
    ],
    max_concurrency=3,  # 并发控制
)
```

**多 Agent 转移**——通过 enum 约束防止 LLM 幻觉：

```js
# 自动限制合法的转移目标
class TransferToAgentTool(FunctionTool):
    def __init__(self, agent_names: list[str]):
        super().__init__(func=transfer_to_agent)
        self._agent_names = agent_names

    def _get_declaration(self):
        decl = super()._get_declaration()
        # 将 agent_name 参数的 schema 限制为 enum
        decl.parameters.properties['agent_name'].enum = self._agent_names
        return decl
```

> 📌 **实践建议**：
>
> + OpenClaw 的"多渠道统一路由"适合需要对接多前端的产品——一套 Agent 逻辑，多端触达
>
> + ADK 的"函数即工具"大幅降低开发门槛：不需要手写 JSON Schema，写个函数就行
>
> + Workflow 图引擎适合有确定性流程的场景（审批链、数据管道），比纯 LLM 路由更可控
>
> + `enum` 约束防幻觉是个巧妙的小技巧：限制模型的选择空间 = 减少犯错概率

* * *

### [](#25-%E6%A8%AA%E5%90%91%E5%AF%B9%E6%AF%94%E6%80%BB%E7%BB%93%E8%A1%A8)2.5 横向对比总结表

| 维度       | Claude Code    | Codex          | Hermes              | OpenClaw        | ADK-Python        |
| -------- | -------------- | -------------- | ------------------- | --------------- | ----------------- |
| 语言/运行时   | TypeScript/Bun | Rust(核心)+TS/Py | Python              | TypeScript      | Python            |
| 架构模式     | 单循环流式          | 沙箱隔离+多Agent    | 单循环+自进化             | 控制平面+路由         | 声明式树+Workflow     |
| **核心亮点** | KV-Cache 感知    | 安全纵深防御         | Prompt Cache 第一公民   | 22+渠道编排         | 函数即工具             |
| 上下文管理    | 五级递进压缩         | 32种片段+Handoff  | 压缩→session 分裂       | 工具结果截断          | State 三级作用域       |
| 工具调度     | 流式执行+智能并发      | 分层编排+审批缓存      | AST发现+8线程并发         | 插件注册表           | 函数自动转Tool         |
| 错误恢复     | withhold+多策略   | 沙箱升级重试         | classifier+fallback | 模型故障转移          | Callback 拦截       |
| 多Agent支持 | SubAgent+Team  | spawn/wait 原语  | delegate\_task      | sessions\_spawn | transfer/workflow |
| 记忆系统     | Compact后文件保留   | 两阶段记忆管道        | 自进化Skill            | 会话持久化           | Memory Service    |
| 安全模型     | 三层权限           | OS级沙箱+网络代理     | 路径防护+injection扫描    | 沙箱+白名单          | 工具确认              |

**一句话总结**：

+ **Claude Code**：省钱省时间（Cache 优化做到极致）

+ **Codex**：安全到骨子里（沙箱是架构而非补丁）

+ **Hermes**：越用越强（自进化 Skill + 零锁定）

+ **OpenClaw**：管理 Agent 的操作系统

+ **ADK-Python**：开发者体验最优（写函数就是造工具）

* * *

## [](#%E4%B8%89%E3%80%81%E4%BB%8E%E9%9B%B6%E5%88%B0-harness%EF%BC%9A%E6%89%8B%E6%8A%8A%E6%89%8B%E6%90%AD%E5%BB%BA%E4%BD%A0%E7%9A%84-agent-%E5%B7%A5%E7%A8%8B)三、从零到 Harness：手把手搭建你的 Agent 工程

> 理论说完了，接下来动手。我们从最简单的 50 行 Agent Loop 开始，逐步添加工具、上下文管理、流式输出、规划和记忆，最终搭建一个完整的 Agent Harness。

### [](#31-%E6%A0%B8%E5%BF%83%E9%AA%A8%E6%9E%B6%EF%BC%9Aagent-loop%EF%BC%8850%E8%A1%8C%E4%BB%A3%E7%A0%81%EF%BC%89)3.1 核心骨架：Agent Loop（50行代码）

Agent 的本质就是一个 **while 循环**——思考、行动、观察、决定是否继续：

```js
"""最小可行 Agent Loop —— 50 行代码"""
import json
from openai import OpenAI

client = OpenAI()

def agent_loop(user_message: str, tools: list[dict], max_steps: int = 10) -> str:
    """最小可行的 Agent 主循环。"""
    messages = [
        {"role": "system", "content": "你是一个有帮助的助手。使用工具来完成任务。"},
        {"role": "user", "content": user_message},
    ]

    for step in range(max_steps):
        # 1. 思考：调用 LLM
        response = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools,
            tool_choice="auto",
        )
        assistant_msg = response.choices[0].message
        messages.append(assistant_msg.model_dump())

        # 2. 判断：是否需要行动？
        if not assistant_msg.tool_calls:
            return assistant_msg.content  # 无工具调用 → 结束

        # 3. 行动：执行工具
        for tool_call in assistant_msg.tool_calls:
            result = execute_tool(tool_call.function.name, tool_call.function.arguments)
            messages.append({
                "role": "tool",
                "tool_call_id": tool_call.id,
                "content": json.dumps(result, ensure_ascii=False),
            })
        # 4. 观察：工具结果已加入上下文，继续循环

    return "达到最大步数限制"  # 兜底
```

**核心模式**：`while True → LLM 决策 → 执行 → 观察 → 继续/结束`。所有复杂的 Agent 都是这个骨架的扩展。

### [](#32-%E5%B7%A5%E5%85%B7%E6%B3%A8%E5%86%8C%E4%B8%8E%E8%B0%83%E5%BA%A6)3.2 工具注册与调度

接下来给 Agent 装上"手脚"——让它能操作文件、执行代码、搜索信息：

```js
"""工具注册中心 —— 声明式 + 自动 Schema 生成"""
import inspect
import json
from typing import Any, Callable

# 全局工具注册表
_TOOL_REGISTRY: dict[str, Callable] = {}


def tool(description: str = ""):
    """装饰器：注册一个函数为 Agent 工具。"""
    def decorator(func: Callable) -> Callable:
        func._tool_description = description or func.__doc__ or ""
        _TOOL_REGISTRY[func.__name__] = func
        return func
    return decorator


def get_tool_schemas() -> list[dict]:
    """自动从注册的函数生成 OpenAI Function Calling Schema。"""
    schemas = []
    for name, func in _TOOL_REGISTRY.items():
        sig = inspect.signature(func)
        properties = {}
        required = []
        for param_name, param in sig.parameters.items():
            if param_name in ("self", "cls"):
                continue
            prop = {"type": _python_type_to_json_type(param.annotation)}
            properties[param_name] = prop
            if param.default is inspect.Parameter.empty:
                required.append(param_name)

        schemas.append({
            "type": "function",
            "function": {
                "name": name,
                "description": func._tool_description,
                "parameters": {
                    "type": "object",
                    "properties": properties,
                    "required": required,
                },
            },
        })
    return schemas


def execute_tool(name: str, arguments: str) -> Any:
    """执行工具并返回结果。"""
    func = _TOOL_REGISTRY.get(name)
    if not func:
        return {"error": f"未知工具: {name}"}
    try:
        args = json.loads(arguments) if isinstance(arguments, str) else arguments
        result = func(**args)
        return {"success": True, "result": result}
    except Exception as e:
        return {"success": False, "error": str(e)}


# ===== 示例工具 =====

@tool("读取指定路径的文件内容")
def read_file(path: str) -> str:
    with open(path, 'r', encoding='utf-8') as f:
        return f.read()


@tool("将内容写入指定路径的文件")
def write_file(path: str, content: str) -> str:
    with open(path, 'w', encoding='utf-8') as f:
        f.write(content)
    return f"文件已写入: {path}"


@tool("在目录中搜索包含关键词的文件")
def search_files(directory: str, keyword: str) -> list[str]:
    import os
    results = []
    for root, dirs, files in os.walk(directory):
        for file in files:
            filepath = os.path.join(root, file)
            try:
                with open(filepath, 'r', encoding='utf-8') as f:
                    if keyword in f.read():
                        results.append(filepath)
            except (UnicodeDecodeError, PermissionError):
                continue
    return results


def _python_type_to_json_type(annotation) -> str:
    """Python 类型注解 → JSON Schema 类型。"""
    type_map = {str: "string", int: "integer", float: "number", bool: "boolean", list: "array"}
    return type_map.get(annotation, "string")
```

### [](#33-%E4%B8%8A%E4%B8%8B%E6%96%87%E7%AA%97%E5%8F%A3%E7%AE%A1%E7%90%86)3.3 上下文窗口管理

当对话越来越长时，如何防止 Token 爆炸？借鉴 Claude Code 的"分级压缩"思路：

```js
"""上下文窗口管理 —— 分级压缩策略"""
from dataclasses import dataclass
from typing import Optional

@dataclass
class ContextConfig:
    max_tokens: int = 128000          # 模型窗口大小
    compress_threshold: float = 0.7    # 70% 触发规则压缩
    summary_threshold: float = 0.9     # 90% 触发摘要压缩
    max_tool_result_tokens: int = 4000 # 单条工具结果上限


class ContextManager:
    """分级上下文管理器。"""

    def __init__(self, config: Optional[ContextConfig] = None):
        self.config = config or ContextConfig()

    def manage(self, messages: list[dict]) -> list[dict]:
        """根据当前 token 使用率执行分级压缩。"""
        usage = self._estimate_usage(messages)

        if usage < self.config.compress_threshold:
            return messages  # 不需要压缩

        if usage < self.config.summary_threshold:
            return self._rule_compress(messages)  # Level 1: 规则压缩（零 LLM 开销）

        return self._summary_compress(messages)  # Level 2: 摘要压缩（需要 LLM 调用）

    def _rule_compress(self, messages: list[dict]) -> list[dict]:
        """Level 1: 规则压缩 —— 截断大型工具结果、移除旧消息。"""
        compressed = []
        for msg in messages:
            if msg["role"] == "system":
                compressed.append(msg)  # System Prompt 永不压缩
                continue
            if msg["role"] == "tool":
                content = msg.get("content", "")
                if len(content) > self.config.max_tool_result_tokens * 4:  # 粗估
                    # 大结果截断 + 标记
                    msg = {**msg, "content": content[:self.config.max_tool_result_tokens * 4]
                           + "\n\n[... 结果已截断，如需完整内容请重新调用工具 ...]"}
            compressed.append(msg)

        # 保留最近 N 条消息 + System Prompt
        if self._estimate_usage(compressed) > self.config.compress_threshold:
            system_msgs = [m for m in compressed if m["role"] == "system"]
            other_msgs = [m for m in compressed if m["role"] != "system"]
            # 保留最近 60% 的消息
            keep_count = max(4, int(len(other_msgs) * 0.6))
            compressed = system_msgs + other_msgs[-keep_count:]

        return compressed

    def _summary_compress(self, messages: list[dict]) -> list[dict]:
        """Level 2: 摘要压缩 —— 用 LLM 生成对话摘要。"""
        from openai import OpenAI
        client = OpenAI()

        system_msgs = [m for m in messages if m["role"] == "system"]
        other_msgs = [m for m in messages if m["role"] != "system"]

        # 将旧消息压缩为摘要
        old_msgs = other_msgs[:-4]  # 保留最近 4 条不压缩
        recent_msgs = other_msgs[-4:]

        if not old_msgs:
            return messages

        summary_response = client.chat.completions.create(
            model="gpt-4o-mini",  # 用小模型做摘要，省钱
            messages=[{
                "role": "system",
                "content": "请将以下对话历史压缩为结构化摘要。保留关键信息：用户意图、已完成操作、重要结果、待处理事项。"
            }, {
                "role": "user",
                "content": json.dumps(old_msgs, ensure_ascii=False)
            }],
        )

        summary = summary_response.choices[0].message.content
        summary_msg = {"role": "user", "content": f"[历史摘要]\n{summary}\n[/历史摘要]"}

        return system_msgs + [summary_msg] + recent_msgs

    def _estimate_usage(self, messages: list[dict]) -> float:
        """估算 token 使用率（粗略：4 字符 ≈ 1 token）。"""
        total_chars = sum(len(json.dumps(m, ensure_ascii=False)) for m in messages)
        estimated_tokens = total_chars / 4
        return estimated_tokens / self.config.max_tokens
```

### [](#34-%E6%B5%81%E5%BC%8F%E8%BE%93%E5%87%BA)3.4 流式输出

让用户实时看到 Agent 的思考过程，而不是等几十秒后突然出一大段：

```js
"""流式输出 —— SSE 模式"""
import asyncio
from typing import AsyncGenerator

async def agent_loop_streaming(
    user_message: str,
    tools: list[dict],
    max_steps: int = 10,
) -> AsyncGenerator[dict, None]:
    """流式 Agent Loop，逐步 yield 事件。"""
    messages = [
        {"role": "system", "content": "你是一个有帮助的助手。"},
        {"role": "user", "content": user_message},
    ]

    for step in range(max_steps):
        yield {"type": "thinking", "content": f"正在思考（第 {step + 1} 步）..."}

        # 流式调用 LLM
        stream = client.chat.completions.create(
            model="gpt-4o",
            messages=messages,
            tools=tools,
            tool_choice="auto",
            stream=True,
        )

        # 逐 chunk 透传给前端
        full_content = ""
        tool_calls_buffer = {}
        for chunk in stream:
            delta = chunk.choices[0].delta
            if delta.content:
                full_content += delta.content
                yield {"type": "text_delta", "content": delta.content}
            if delta.tool_calls:
                for tc in delta.tool_calls:
                    idx = tc.index
                    if idx not in tool_calls_buffer:
                        tool_calls_buffer[idx] = {"id": tc.id, "name": "", "arguments": ""}
                    if tc.function.name:
                        tool_calls_buffer[idx]["name"] = tc.function.name
                    if tc.function.arguments:
                        tool_calls_buffer[idx]["arguments"] += tc.function.arguments

        if not tool_calls_buffer:
            yield {"type": "done", "content": full_content}
            return

        # 执行工具
        for idx, tc_info in tool_calls_buffer.items():
            yield {"type": "tool_start", "tool": tc_info["name"], "args": tc_info["arguments"]}
            result = execute_tool(tc_info["name"], tc_info["arguments"])
            yield {"type": "tool_result", "tool": tc_info["name"], "result": result}
            messages.append({
                "role": "tool",
                "tool_call_id": tc_info["id"],
                "content": json.dumps(result, ensure_ascii=False),
            })

    yield {"type": "max_steps", "content": "达到最大步数限制"}
```

### [](#35-%E8%BF%9B%E9%98%B6%EF%BC%9A%E5%8A%A0%E5%85%A5-planning-memory)3.5 进阶：加入 Planning + Memory

#### [](#%E7%AE%80%E6%98%93-planning%EF%BC%9Atodomd-%E6%A8%A1%E5%BC%8F)简易 Planning：todo.md 模式

参考 Manus 和 Codex 的做法——用一个简单的 Markdown 文件作为"任务计划"：

```js
"""Planning 模块 —— todo.md 模式"""

PLAN_TOOL = {
    "type": "function",
    "function": {
        "name": "update_plan",
        "description": "创建或更新任务计划。计划应包含多个步骤，不允许只有一步。",
        "parameters": {
            "type": "object",
            "properties": {
                "steps": {
                    "type": "array",
                    "items": {
                        "type": "object",
                        "properties": {
                            "id": {"type": "integer"},
                            "description": {"type": "string"},
                            "status": {"type": "string", "enum": ["pending", "in_progress", "done", "failed"]},
                        },
                    },
                },
            },
            "required": ["steps"],
        },
    },
}


class PlanManager:
    """管理 Agent 的执行计划。"""

    def __init__(self):
        self.steps: list[dict] = []

    def update(self, steps: list[dict]) -> str:
        """更新计划。"""
        self.steps = steps
        return self._render()

    def mark_step(self, step_id: int, status: str) -> str:
        """标记步骤状态。"""
        for step in self.steps:
            if step["id"] == step_id:
                step["status"] = status
                break
        return self._render()

    def _render(self) -> str:
        """渲染为 Markdown 格式。"""
        lines = ["## 📋 执行计划\n"]
        status_icons = {"pending": "⬜", "in_progress": "🔄", "done": "✅", "failed": "❌"}
        for step in self.steps:
            icon = status_icons.get(step["status"], "⬜")
            lines.append(f"{icon} {step['id']}. {step['description']}")
        return "\n".join(lines)

    def get_progress_summary(self) -> str:
        """生成进度摘要，注入到上下文中。"""
        total = len(self.steps)
        done = sum(1 for s in self.steps if s["status"] == "done")
        current = next((s for s in self.steps if s["status"] == "in_progress"), None)
        summary = f"进度: {done}/{total} 完成"
        if current:
            summary += f" | 当前步骤: {current['description']}"
        return summary
```

#### [](#%E7%AE%80%E6%98%93-memory%EF%BC%9A%E6%96%87%E4%BB%B6%E7%B3%BB%E7%BB%9F%E4%BD%9C%E4%B8%BA%E6%89%A9%E5%B1%95%E8%AE%B0%E5%BF%86)简易 Memory：文件系统作为扩展记忆

```js
"""Memory 模块 —— 文件系统持久化"""
import os
import json
from datetime import datetime

class FileMemory:
    """基于文件系统的持久记忆。"""

    def __init__(self, memory_dir: str = ".agent_memory"):
        self.memory_dir = memory_dir
        os.makedirs(memory_dir, exist_ok=True)

    def save_session_summary(self, session_id: str, summary: str) -> None:
        """保存会话摘要。"""
        filepath = os.path.join(self.memory_dir, f"session_{session_id}.json")
        data = {
            "session_id": session_id,
            "summary": summary,
            "timestamp": datetime.now().isoformat(),
        }
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(data, f, ensure_ascii=False, indent=2)

    def recall_recent(self, limit: int = 5) -> list[dict]:
        """回忆最近的会话摘要。"""
        memories = []
        for filename in sorted(os.listdir(self.memory_dir), reverse=True):
            if filename.startswith("session_") and filename.endswith(".json"):
                filepath = os.path.join(self.memory_dir, filename)
                with open(filepath, 'r', encoding='utf-8') as f:
                    memories.append(json.load(f))
                if len(memories) >= limit:
                    break
        return memories

    def save_learning(self, key: str, content: str) -> None:
        """保存 Agent 学到的经验。"""
        filepath = os.path.join(self.memory_dir, "learnings.json")
        learnings = {}
        if os.path.exists(filepath):
            with open(filepath, 'r', encoding='utf-8') as f:
                learnings = json.load(f)
        learnings[key] = {"content": content, "timestamp": datetime.now().isoformat()}
        with open(filepath, 'w', encoding='utf-8') as f:
            json.dump(learnings, f, ensure_ascii=False, indent=2)
```

### [](#36-harness-%E5%B7%A5%E7%A8%8B%E5%8C%96%EF%BC%9A%E4%BB%8E%E7%8E%A9%E5%85%B7%E5%88%B0%E6%AD%A6%E5%99%A8)3.6 Harness 工程化：从玩具到武器

> 前面 3.1-3.5 教你造了一辆"能跑的车"。这一节教你造一整套"赛道+测试场+维保系统"，也就是 Harness 工程。

#### [](#%E4%BB%80%E4%B9%88%E6%98%AF-agent-harness%EF%BC%9F)什么是 Agent Harness？

![](https://km.woa.com/asset/000100022606002005911d2571433e01?height=1024&width=1536)

+ **定义**：包裹在 Agent 外层的工程化脚手架——负责配置管理、工具编排、评估驱动、可观测性

+ **类比**：Agent 是发动机，Harness 是整车工程（底盘+仪表盘+测试跑道）

+ **没有 Harness 的痛**：每次调试靠"跑一个问题看一下"→ 无法回归 → 无法量化改进 → 永远在"玄学调参"

+ **有 Harness 的爽**：配置改一行 → 自动跑 100 个测试用例 → 5分钟出评估报告 → 精准定位问题

#### [](#%E6%8E%A8%E8%8D%90%E9%A1%B9%E7%9B%AE%E7%9B%AE%E5%BD%95%E7%BB%93%E6%9E%84)推荐项目目录结构

```js
my-agent-harness/
├── agent/
│   ├── core.py              # Agent Loop 主循环
│   ├── planner.py           # Planning 模块
│   └── router.py            # 多 Agent 路由（进阶）
├── tools/
│   ├── registry.py          # 工具注册中心
│   ├── builtin/             # 内置工具实现
│   └── mcp_client.py        # MCP 协议客户端（可选）
├── context/
│   ├── manager.py           # 上下文窗口管理
│   ├── compressor.py        # 压缩策略
│   └── prompt_templates/    # Prompt 模板目录
├── memory/
│   ├── short_term.py        # 短期记忆
│   └── long_term.py         # 持久记忆
├── eval/
│   ├── runner.py            # 评测运行器
│   ├── judge.py             # 自动评判（LLM-as-Judge）
│   └── cases/              # 黄金测试用例（YAML）
│       ├── basic.yaml
│       └── edge_cases.yaml
├── config/
│   ├── models.yaml          # 模型配置（主模型/降级模型/路由规则）
│   ├── tools.yaml           # 工具配置
│   └── prompts.yaml         # Prompt 配置
├── scripts/
│   ├── run_eval.sh          # 一键跑评测
│   └── deploy.sh            # 部署脚本
├── main.py                  # 入口
└── README.md
```

#### [](#%E5%85%B3%E9%94%AE%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99)关键设计原则

从 Claude Code Codex Hermes / Dola 中提炼的 **6 条通用法则**：

| #   | 原则              | 说明                                             | 来源                 |
| --- | --------------- | ---------------------------------------------- | ------------------ |
| 1   | **配置与代码分离**     | 模型、Prompt、工具列表全部外置，改配置不改代码                     | Hermes/Codex       |
| 2   | **Prompt 分层管理** | System Prompt = 角色 + 能力边界 + 工具说明 + 格式约束，各层独立维护 | Claude Code        |
| 3   | **工具标准化**       | 统一 Schema（OpenAI FC 格式或 MCP），新增工具只需加配置         | ADK/Hermes         |
| 4   | **上下文预算制**      | 每类信息分配 Token 预算，超预算自动压缩而非"塞到满"                 | Claude Code        |
| 5   | **评估即开发**       | 先写测试用例，再改 Agent——评估驱动开发（EDD）                   | Codex              |
| 6   | **错误即数据**       | 所有失败保留在日志中，作为后续优化的"训练数据"                       | Claude Code/Hermes |

#### [](#%E4%BA%94%E6%AD%A5%E6%90%AD%E5%BB%BA%E4%BD%A0%E7%9A%84%E7%AC%AC%E4%B8%80%E4%B8%AA-harness)五步搭建你的第一个 Harness

```js
Step 1: 先跑通 Agent Loop（3.1 的 50 行代码）
         ↓ 验证：能调通模型、能手动输入问题得到回答
Step 2: 加工具注册 + 2-3 个核心工具
         ↓ 验证：Agent 能自主调用工具并利用结果回答
Step 3: 加配置文件（models.yaml + prompts.yaml），实现配置化
         ↓ 验证：改配置即可切换模型/Prompt，无需改代码
Step 4: 写 5-10 个黄金测试用例（YAML），加评测脚本
         ↓ 验证：一键跑评测、输出通过率报告
Step 5: 加上下文压缩 + 记忆，进入"改配置→跑评测→看报告"循环
         ↓ 验证：长对话不崩、跨会话有记忆
```

> 💡 **核心心法**：不要一开始就追求完美架构。先跑通最小闭环（Agent + 工具 + 评测），再逐步加层。每加一层都有评测兜底，确保不退化。

#### [](#%E6%B5%8B%E8%AF%95%E7%94%A8%E4%BE%8B%E6%A8%A1%E6%9D%BF)测试用例模板

```js
# eval/cases/basic.yaml
- id: "test_001"
  question: "读取 README.md 文件并总结其内容"
  expected:
    tool_used: ["read_file"]
    answer_contains: ["README", "项目"]
  tags: ["basic", "file_read"]

- id: "test_002"
  question: "在 src/ 目录中查找包含 'TODO' 的文件，并列出它们"
  expected:
    tool_used: ["search_files"]
    answer_type: "list"
  tags: ["basic", "search"]

- id: "test_003"
  question: "创建一个新文件 hello.py，内容为打印 Hello World"
  expected:
    tool_used: ["write_file"]
    file_created: "hello.py"
    file_contains: ["print", "Hello"]
  tags: ["basic", "file_write"]
```

#### [](#%E5%B8%B8%E8%A7%81%E8%B8%A9%E5%9D%91%E4%B8%8E%E9%81%BF%E5%9D%91%E6%8C%87%E5%8D%97)常见踩坑与避坑指南

| 坑        | 表现               | 解法                                             |
| -------- | ---------------- | ---------------------------------------------- |
| Token 爆炸 | 对话几轮后模型开始胡说      | 上下文预算制 + 分级压缩（3.3）                             |
| 工具幻觉     | 模型编造不存在的工具名      | 工具描述精简 + 严格 Schema + enum 约束                   |
| 无限循环     | Agent 反复调同一个工具   | max\_steps 保护 + Stuck 检测（连续3次相同调用则终止）          |
| 评测不稳定    | 同一问题每次答案不同       | temperature=0 + 多次采样 + 宽松匹配规则                  |
| 配置混乱     | 不知道当前用的哪版 Prompt | 配置版本化 + 变更日志 + 评测快照                            |
| 错误吞没     | 工具报错但 Agent 假装成功 | 保留 error 在上下文（参考 Claude Code 的 is\_error:true） |

* * *

## [](#%E5%9B%9B%E3%80%81%E4%BB%8E-demo-%E5%88%B0%E7%94%9F%E4%BA%A7%E7%BA%A7%EF%BC%9A%E5%85%B3%E9%94%AE%E5%B7%AE%E8%B7%9D)四、从 Demo 到生产级：关键差距

Demo 级 Agent 只需 100 行代码，但**生产级**要解决的问题多得多：

| 维度   | Demo 级   | 生产级                         |
| ---- | -------- | --------------------------- |
| 并发   | 同步阻塞     | 异步 + 连接池 + 限流               |
| 容错   | 一报错就挂    | 自动重试 + 降级 + 熔断              |
| 可观测性 | print 调试 | trace\_id 串联 + 结构化日志 + 监控面板 |
| 效果评估 | 人肉看输出    | 自动化评测 + CI 集成 + 回归检测        |
| 成本控制 | 不管花多少    | Token 预算 + Cache 优化 + 模型路由  |
| 安全性  | 无防护      | 沙箱隔离 + 权限控制 + injection 防护  |

**从 Demo 到生产的关键一步**：建立"改 Prompt/工具 → 跑评测 → 看报告 → 确认无退化 → 上线"的工程闭环。没有这个闭环，你永远在"玄学调参"。

* * *

## [](#%E4%BA%94%E3%80%81%E6%80%BB%E7%BB%93%E4%B8%8E%E5%B1%95%E6%9C%9B)五、总结与展望

### [](#%E6%A0%B8%E5%BF%83%E8%AE%BE%E8%AE%A1%E5%8E%9F%E5%88%99%E5%9B%9E%E9%A1%BE)核心设计原则回顾

从五个顶级 Coding Agent 中，我们提炼出以下**跨项目共识**：

1. **上下文工程 > Prompt 工程**——不是写一个好 Prompt，而是设计一套信息管理系统

2. **Cache/成本感知应是第一公民**——Claude Code 和 Hermes 都将 Prompt Cache 作为核心架构约束

3. **错误即数据，失败即学习**——保留错误历史让模型自我修正，比隐藏错误强100倍

4. **工具边界显式化**——`isConcurrencySafe`、`require_confirmation`、审批缓存……让系统知道每个工具的"脾气"

5. **安全是架构而非补丁**——Codex 用 OS 级沙箱证明了这一点

6. **评估驱动开发**——先有测试用例，再有 Agent 改进

### [](#%E5%8A%A8%E6%89%8B%E6%8C%91%E6%88%98)动手挑战

如果你读完本文想动手实践，这里有一个渐进式挑战清单：

+ **Level 1**：实现 3.1 的 50 行 Agent Loop，能和 LLM 对话

+ **Level 2**：加 3 个工具（读文件/写文件/搜索），Agent 能自主完成文件操作

+ **Level 3**：加上下文压缩，让 Agent 支持 20+ 轮长对话不崩

+ **Level 4**：加 Planning 模块，Agent 能自主拆解多步任务

+ **Level 5**：加评测系统，实现"改配置→跑评测→看报告"闭环

+ **Boss**：加 Skill 沉淀，Agent 完成任务后自动生成可复用经验

## [](#%E5%8F%82%E8%80%83%E6%BA%90%E7%A0%81)参考源码

| 项目           | 地址                                                                             | 核心亮点                  |
| ------------ | ------------------------------------------------------------------------------ | --------------------- |
| Claude Code  | [github.com/anthropics/claude-code](https://github.com/anthropics/claude-code) | KV-Cache 感知 + 五级压缩    |
| OpenAI Codex | [github.com/openai/codex](https://github.com/openai/codex)                     | OS级沙箱 + 多Agent原语      |
| Hermes Agent | [github.com/hermes-agent](https://github.com/)                                 | AST工具发现 + 自进化Skill    |
| OpenClaw     | [github.com/openclaw](https://github.com/)                                     | 控制平面 + Prompt as Code |
| Google ADK   | [github.com/google/adk-python](https://github.com/google/adk-python)           | 函数即工具 + Workflow引擎    |
