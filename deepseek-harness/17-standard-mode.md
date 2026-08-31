---
seoTitle: "DeepSeek Harness Standard Mode：完整编码 Agent 模式"
description: "本篇将验证 Standard Mode 的定位、默认工具和适用任务，使用固定版本记录真实工具暴露与运行过程，并为后续四模式对比建立同一任务基线。"
lastVerified: "2026-08-29"
author: stormzhang
dshVersion: "0.1.1-rc.2"
verificationLevel: "V1"
verificationStatus: "preset-runtime-and-real-provider-comparison-passed-owner-experience-confirmed"
stability: "version-bound"
officialSources:
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/README.zh.md"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/packages/bundle/base/README.zh.md"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/packages/core/agent-loop/README.zh.md"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/packages/core/tools/README.zh.md"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/apps/cli/config/agent-presets/standard/agent.cordis.yml"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/apps/cli/config/agent-presets/standard/preset.yml"
related:
  - "./18-ptc-code-mode.md"
  - "./21-mode-selection.md"
evidence:
  - "references/deepseek-harness/ARTICLE-SOURCE-MAP.md：第 17 篇"
  - "references/deepseek-harness/RUNTIME-BASELINE.md"
  - "2026-08-27 北京时间复核固定快照与 master 的 Standard Preset 文件 SHA 一致"
---

# 17 · Standard Mode：完整编码 Agent

> 📚 **系列导航**：上一篇 [16 · 怎么给 Harness 下达任务](./16-task-instructions.md) 已经把目标、范围和验证写成执行合同。这一篇进入默认的 Standard Mode，看看模型收到的 25 个原生工具怎样把合同变成行动。

> [!WARNING] Developer Preview
> 固定版已实测 `standard` Preset 的 25 个工具，并用真实 Provider 完成同任务基线，工具选择、耗时、usage 量级、文件变化与测试均已核对。

固定版 Standard Mode 向模型暴露 25 个工具。

这个数字不代表它一定比只有一个 `run_code` 的 PTC 更强，也不代表一次任务会把 25 个工具全用一遍。

它真正的含义是：**模型可以直接看到每个原生工具的名称、描述和参数 schema，并逐个发出 Tool Call。**

**看完这一篇，你会拿到：**

- 认准 Standard 的 UI 名称、Preset ID 与默认位置
- 看懂固定版 25 个原生工具的完整分组
- 理解 Native Tool Call 怎样进入 Agent Loop
- 知道并行安全调用与独占调用如何调度
- 判断哪些任务优先使用 Standard
- 明白 Preset 为什么只在新建 Session 时选择
- 用统一任务建立 Standard 的 Native 对照基线

---

## 01 先认准 Standard 的真实身份

固定版随附四个系统 Preset，其中 Standard 的事实是：

```text
UI 名称：标准模式
Preset ID：standard
排序：1
默认 Preset：是
工具呈现：native
固定版 macOS 工具数：25
```

官方组装对它的描述是「功能完整的编码 Agent，支持文件编辑、Shell、文件与网页检索、Skills、计划、目标、子代理和工作流」。

| 名称 | 用在哪里 | 是否稳定身份 |
|---|---|---|
| 标准模式 | 中文 Web UI 展示 | 否，文案可调整 |
| Standard Mode | 教程称呼 | 否，便于沟通 |
| `standard` | Session 与配置记录 | 是，固定版真实 ID |

**类比：Preset 像一套岗位配置。** `standard` 是岗位编号，「标准模式」是人类看到的职位名称，25 个工具是这名员工上岗时领到的工具箱。

G02 隔离实测中，`agentPreset.list` 把 `standard` 标记为默认项；使用它创建的 Session 正常完成 Mock Turn 和 `turn/end`。

2026 年 8 月 29 日，本教程第一次用真实 Provider 跑 Standard 基线，任务仍是修复 `calculator.js` 里写反的加法。Provider 为 `deepseek-official`，模型为 `deepseek-v4-flash`；它直接使用 `bash`、`read`、`edit` 等原生工具，5 个 Step 完成任务。最终只改一行实现，独立测试 1 通过、0 失败，没有审批，也没有碰测试文件。

> 💡 **一句话总结**：界面叫「标准模式」，持久身份是 `standard`，它是固定版新 Session 的默认完整编码组合。

---

## 02 Native Tool Call 到底是什么

Standard 使用原生工具呈现（Native Tool Calling）。模型请求中会直接带上每个可见工具的：

- 工具名。
- 用途描述。
- JSON Schema 参数。

模型需要读文件时，直接发出 `read`；需要搜索时发出 `grep`；需要跑测试时发出 `bash`。Harness 不要求它先包一层程序。

```text
模型请求
→ read Tool Call
→ read Tool Result
→ 下一 Step 模型请求
→ edit Tool Call
→ edit Tool Result
→ 下一 Step 模型请求
→ bash Tool Call
→ bash Tool Result
```

| Native 的优点 | Native 的代价 |
|---|---|
| 每次调用意图直观 | 多步依赖任务可能需要更多模型往返 |
| 审批与工具卡容易检查 | 每个请求都要携带可见工具 schema |
| 错误对应到具体工具 | 中间结果会进入后续模型上下文 |
| 适合边看结果边决策 | 大量机械操作可能显得啰嗦 |

Native 不等于「每个 Step 只能一个工具」。模型可以在同一回复中提出多个调用，Agent Loop 会根据工具声明区分并行安全与独占调用。

Standard 的 25 个直接 schema 与 PTC 单一 `run_code` 入口如何落到同一套底层工具与安全管线，见下一篇 [18 · PTC 模式（Code Mode）](./18-ptc-code-mode.md) 的「Native 与 Code Mode 工具呈现对照」图。

> 💡 **一句话总结**：Native 让模型直接点名真实工具，过程透明，但多步任务可能增加模型--工具往返。

---

## 03 固定版 25 个工具完整清单

G02 捕获的 Standard 模型请求实际包含 25 个工具。它们可以按用途分成七组：

| 分组 | 工具 | 数量 |
|---|---|---:|
| 用户与计划 | `ask_user_question`、`exit_plan_mode` | 2 |
| Shell、文件与搜索 | `bash`、`read`、`read_image`、`write`、`edit`、`glob`、`grep` | 7 |
| Goal 与待办 | `create_goal`、`get_goal`、`update_goal`、`todo_write` | 4 |
| Skills 与网页 | `skill`、`web_search` | 2 |
| 后台任务 | `job_list`、`job_output`、`job_kill` | 3 |
| Subagent | `subagent`、`subagent_fork`、`list_agents`、`send_message`、`interrupt_agent` | 5 |
| Workflow | `workflow`、`ralph` | 2 |
| **合计** |  | **25** |

有四个边界需要单独说明：

1. Windows 组合用 `pwsh` 替换 `bash`，不是同时提供两个 Shell。
2. 固定版 Standard 的 Web 配置关闭 `fetch`，因此本次只有 `web_search`，没有 `web_fetch`。
3. `exit_plan_mode` 即使当前不在 Plan Mode 也保留 schema，用于保持工具目录稳定；在错误状态调用会被拒绝。
4. 可见工具由 Preset、作用域、平台与已加载插件共同决定，未来版本不能照抄「永远 25 个」。

工具多也不等于每项都该在普通任务中使用。修一个加法 Bug，不需要 Goal、Subagent、Workflow 或网页搜索。

> 💡 **一句话总结**：25 是固定版 macOS Standard 的真实请求计数，不是永久产品规格，更不是每轮必用清单。

---

## 04 一次 Standard 任务怎样推进

以 `first-task` 修复为例，Native 流程可能是：

```text
Step 1：read package.json、calculator.js、calculator.test.js
Step 2：edit calculator.js
Step 3：bash 执行 npm test
Step 4：生成最终交付
```

实际 Step 数不固定。模型也可能在第一步并行读取三个文件，或者读取后同一回复继续提出编辑调用。

Agent Loop 的调度原则是：

- 连续的并行安全调用进入有界滚动池。
- 独占调用先等待已有并行调用排空，再单独执行。
- 独占调用构成排序屏障，后面的调用不能越过它。
- 策略、持久 Tool Result 与上下文仍保持模型顺序。

固定版 `maxParallelToolCalls` 默认是 10，但这只是每个 Agent 的最大池大小。**默认 10 不代表十个调用一定同时运行。** 工具必须明确返回并行安全，只有确切的 `true` 才能重叠执行。

| 调用关系 | 推荐表达 |
|---|---|
| 三个互不依赖的只读文件 | 可以同一 Step 提出 |
| 先读文件再按内容编辑 | 必须先拿到读取结果 |
| 编辑后再运行测试 | 按顺序执行 |
| 两个会修改同一文件的调用 | 不应并行 |

> 💡 **一句话总结**：Standard 可以并行独立读取，但依赖链和写操作仍要按真实结果顺序推进。

---

## 05 Standard 最适合哪些任务

优先选择 Standard 的典型场景：

- 第一次探索陌生仓库，需要边读边决定下一步。
- 修 Bug，需要根据每次测试输出调整方案。
- 修改范围小，希望每个 Tool Call 都容易检查。
- 任务可能触发审批，需要清楚看到具体命令。
- 需要临时调用 Skill、Goal、Subagent 或 Workflow。

| 任务 | Standard 判断 | 原因 |
|---|---|---|
| 读取三份文件并解释差异 | 合适 | 原生读取透明，可能并行 |
| 修复一个失败测试 | 合适 | 适合读--改--测循环 |
| 批量处理大量独立文件 | 可以，但未必最高效 | 可能产生很多往返 |
| 已知流程的机械化多步操作 | 可用，后续可对照 PTC | PTC 可能用一个程序编排 |
| 只想评测最小工具调用 | 不优先 | Minimal 更适合隔离变量 |
| 编写或修改 Preset／插件组合 | 能做，但 Creator 更聚焦 | Standard 没有 Cordis 专属工具 |

Standard 的核心优势不是「最强」，而是**通用、透明、默认可用**。不知道选哪个时，从 Standard 开始最稳妥。

这个单文件 Bug 很适合 Standard：模型先读、再测、再改、再测，一共 5 个 Step、7 次外层工具调用，墙钟约 9 秒，轨迹几乎就是人类排查顺序。后来本教程用 Minimal 重跑同一任务，虽然也成功，却增加到 11 个 Step、约 18 秒。这个结果没有证明 Standard 永远最好，但让我在普通读改测任务里继续把它当默认值，不为了模式新鲜感主动切换。

> 💡 **一句话总结**：Standard 是通用默认，不是所有任务的性能上限；先用透明模式建立基线，再判断是否需要换。

---

## 06 Preset 只能在新 Session 中选择

Web UI 的 Preset chip 位于新建会话界面。选择值只应用到下一个空白 Session，一旦该 Session 已经开始产生内容，Preset 就锁定。

原因很直接：Session 历史是在原 Preset 的系统提示词和工具集合下产生的。中途换成另一套工具呈现，会让后续请求与之前的 Tool Call 历史失去协议连续性。

```text
新建空白 Session：可以从 standard 改为 code
已经发送任务的 Session：不能改 Preset
更改全局默认：只影响后续新 Session
现有 Session：继续使用创建时的 Preset
```

| 想做的事 | 正确操作 |
|---|---|
| 下个任务改用 PTC | 新建 Session，选择 PTC 模式 |
| 已运行任务中途改模式 | 停止后新建 Session，不强行切换 |
| 以后默认都用 Standard | 设置里修改新会话默认值 |
| 对照两个模式 | 两个独立 Session，使用一致项目副本 |

界面标题旁的 Preset 标签是只读身份，不是切换按钮。

> 💡 **一句话总结**：Preset 属于 Session 创建事实，不能在已经产生工具历史后热切换。

---

## 07 动手：建立 Standard 同任务基线

准备一个全新的 `first-task-standard` 项目副本，内容与第 11 篇起始状态一致。进入目录后确认：

```bash
npm test
```

预期为：

```text
add returns sum
actual: -1
expected: 5
fail 1
```

在 Web UI 添加该 Workspace，新建 Session：

```text
Preset：标准模式（standard）
Permission：Workspace Write
Provider／Model：真实对照使用的固定组合
Reasoning：固定同一档位
```

发送统一任务：

```text
修复当前项目唯一的测试失败。

先读取 package.json、calculator.js 和 calculator.test.js。只修改 calculator.js，不修改测试、package.json，不新增依赖。修改后实际运行 npm test，失败就根据真实输出继续修复。

最终汇报根因、修改文件、测试计数和退出结果。未运行的验证不得写成已通过。
```

任务结束后独立执行：

```bash
npm test
```

成功标准：

```text
calculator.js：return a + b
修改文件：只有 calculator.js
测试：1 passed，0 failed
退出码：0
Trajectory：存在直接的 read／edit／bash Tool Call 与对应结果
```

> 💡 **一句话总结**：Standard 基线要固定项目、模型、权限和提示词，并保留直接 Native Tool Call 证据。

---

## 08 应该记录哪些对照数据

为了下一篇与 PTC 公平比较，记录以下数据：

| 指标 | Standard 记录 |
|---|---|
| Provider／Model／Reasoning | 待测 |
| 最终测试 | 待测 |
| 修改文件 | 待测 |
| Turn 数 | 待测 |
| Step 数 | 待测 |
| 直接 Tool Call 数 | 待测 |
| 每种工具次数 | 待测 |
| 审批次数 | 待测 |
| 总耗时 | 待测 |
| TTFT | 待测 |
| 输入／输出 Token | 待测 |
| 是否需要人工纠正 | 待测 |

不要把 Mock Provider 的 Token 和耗时放进对照结论。G02 的 Mock Turn 只证明 Standard 请求确实携带 25 个工具，Agent Loop 与 Session 事件可运行。

同样，不要因为 Standard 多了几个 Step 就提前宣布 PTC 更快。PTC 的模型生成程序、worker 执行和外层结果同样有成本，必须看真实测量。

完整基线使用 `deepseek-official`、`deepseek-v4-flash` 和 `reasoning: high`。Standard 共 5 个 Step，模型直接调用 7 次工具，外层顺序是 `bash`、`read`×3、`edit`、`bash`；墙钟约 9 秒，新鲜输入约 9k 量级，没有触发审批，只修改 `calculator.js`，最终测试 1/1。TTFT 没有在这组记录里单独固化，所以我不补一个估算值；现有数据足以作为后续模式对照基线。

> 💡 **一句话总结**：先完整记录 Standard，下一篇的 PTC 比较才有可信基准。

---

## 09 常见误判与排错

| 现象 | 真实含义 | 优先处理 |
|---|---|---|
| 模型没用某个工具 | 可见不等于必须调用 | 检查任务是否真的需要 |
| 工具数不是 25 | 版本、平台、插件或作用域不同 | 看请求头与实际 schema |
| Windows 看不到 `bash` | 固定组合使用 `pwsh` | 使用 PowerShell 方言 |
| 没有 `web_fetch` | Standard 固定配置关闭 fetch | 不把 `web_search` 当网页抓取 |
| `exit_plan_mode` 在列表里 | 为目录稳定保留 | 非 Plan Mode 调用会拒绝 |
| 多个读取没有并行 | 工具或参数未声明安全 | 不强求模型制造并发 |
| 已运行 Session 无法换 PTC | Preset 已锁定 | 新建 Session |
| 最终回答说测试通过 | 只是模型报告 | 查 `bash` Tool Result 并独立重跑 |

工具目录变化属于高频版本事实。升级 Harness 后，先在 Trajectory 或真实请求证据中重新计数，不要把本文 25 个名称当永久接口。

> 💡 **一句话总结**：以当前 Session 的实际工具 schema 和 Tool Result 为准，不以教程数字倒推运行时。

---

## 小结

这一篇建立了 Standard Mode 的真实基线：

1. UI 名称是「标准模式」，Preset ID 是 `standard`。
2. 固定版将 Standard 作为默认新会话组合。
3. Native 模式让模型直接看到并调用每个工具 schema。
4. G02 在 macOS 实测 25 个工具，Windows 以 `pwsh` 替换 `bash`。
5. 并行安全调用可进入有界池，独占调用构成顺序屏障。
6. Standard 适合通用、透明、需要边看结果边决策的任务。
7. Preset 在 Session 创建后锁定，对照实验必须新开 Session。
8. 真实 Provider 的 Standard 任务基线已完成，5 个 Step、7 次工具调用、约 9 秒，最终测试 1/1。

你现在应该已经知道 Standard 为什么是默认模式，也能从 25 个工具中判断一次任务真正需要哪些能力。

---

下一篇：[18 · PTC 模式（Code Mode）](./18-ptc-code-mode.md)。工具能力不变，但模型看到的入口会从 25 个 Native schema 收敛成一个 `run_code`。
