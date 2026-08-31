---
seoTitle: "DeepSeek Harness、Claude Code、Codex、OpenCode 怎么选：一篇讲清"
description: "同日在线复核四家官方资料，从定位、模型绑定、架构开放性、权限安全、获取方式五个维度对比 DeepSeek Harness、Claude Code、Codex、OpenCode，给出按人划分的选型决策路径。"
lastVerified: "2026-08-31"
author: stormzhang
dshVersion: "0.1.1-rc.2"
verificationLevel: "V4"
verificationStatus: "competitor-facts-rechecked-online-2026-08-31-owner-experience-confirmed"
stability: "live-external"
officialSources:
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/README.zh.md"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/docs/glossary.zh.md"
  - "https://code.claude.com/docs/en/overview"
  - "https://code.claude.com/docs/en/hooks"
  - "https://learn.chatgpt.com/docs/codex/cli"
  - "https://learn.chatgpt.com/docs/hooks"
  - "https://learn.chatgpt.com/docs/open-source"
  - "https://opencode.ai/docs/"
related:
  - "./02-agent-model-harness.md"
  - "./04-model-agent-harness-ide.md"
evidence:
  - "references/deepseek-harness/ARTICLE-SOURCE-MAP.md：第 03 篇"
  - "references/deepseek-harness/COMPARE-BASELINE.md：第 2～5、8 节"
---

# 03 · DeepSeek Harness、Claude Code、Codex、OpenCode 怎么选

> 📚 **系列导航**：上一篇 [02 · Agent = Model + Harness](./02-agent-model-harness.md) 立住了「模型 + Harness」的公式。这一篇回答每个读者都会问的问题：市面上这么多工具，我到底该用哪个？

> [!WARNING] 实时外部事实
> 本文固定 `0.1.1-rc.2`。竞品事实已于 2026-08-31（北京时间）在线核对：DeepSeek Harness 最新 Release 为 `dsh-v0.1.2-alpha.2`（npm 仅 `alpha` 通道提供，`latest` 仍为 `0.1.1-rc.2`）；Claude Code hooks 33 项、Codex hooks 11 项。竞品信息按日过期，引用前请当日重查官方页面。

先说句实话：这是全系列最容易写砸的一篇。

写成「 Harness 天下第一」是广告，写成「四家差不多」是废话，照抄各自官网的功能清单是翻译。本篇不做这三件事。

上一篇咱们已经立住了公式：**Agent = Model + Harness，模型决定能力上限，Harness 决定模型能否在真实环境中稳定完成工作。** 这一篇就用这个公式当尺子，把四个产品放到同一张桌子上量一遍。所有竞品结论都来自 2026-08-31 当日在线核对的官方资料，不凭印象说话。

**看完这一篇，你会拿到：**

- 一张四家产品的定位地图：它们其实不完全是同一层的东西
- 五个真正影响选择的对比维度，每个维度都有当日核对的事实
- 一条「什么人选什么」的决策路径，不拉踩，只匹配

---

## 01 先摆正：这四个不完全是同类

直接对比之前，得先承认一件事：**这四个名字不全是同一层的产品。**

| 产品 | 一句话定位 | 本质 |
|---|---|---|
| Claude Code | Anthropic 的官方编码 Agent 产品 | 模型厂商出品的完整产品（模型 + Harness 打包卖） |
| Codex | OpenAI 的官方编码 Agent 产品 | 完整产品体系；CLI、SDK 与 App Server 开源，IDE 扩展和云端产品不开源 |
| OpenCode | 开源的多模型编码 Agent | 开放的 Harness + 自带任意模型 key |
| DeepSeek Harness | DeepSeek 开源的 Agent Harness | Agent 基础设施／运行时，MIT 协议，Developer Preview |

Claude Code 和 Codex 都提供可直接工作的「成品车」，但开放边界并不相同：Codex 的 CLI、SDK 与 App Server 代码公开，IDE 扩展和云端产品不公开。OpenCode 是开源的多模型成品，模型 Key 自己配置。DeepSeek Harness 更强调「底盘制造图纸 + 一条示范生产线」：它把 Agent 运行时和插件化改造本身放到产品中心。

**类比：买手机。** Claude Code 更接近软硬件一体的成品；Codex 和 OpenCode 都开放了本地工具链，只是账号、模型与云端边界不同；DeepSeek Harness 更像把系统运行时和扩展框架一起交给你。这个类比只描述开放层级，不代表质量排名。

所以本篇真正回答的问题不是「哪个最强」，而是「你要的是成品、可换模型的成品，还是可改造的基础设施」。

> 💡 **一句话总结**：四个产品分属「打包成品」和「开放基础设施」两层，先想清楚自己要哪一层，再谈功能对比。

---

## 02 获取方式与门槛（2026-08-30 当日核对）

选型的第一道筛子往往不是功能，而是「我能不能用、怎么付钱」。以下为当日在线核对结果：

| 产品 | 获取路径 | 账号与费用门槛 |
|---|---|---|
| DeepSeek Harness | `npx @deepseek-ai/dsh web` 或源码构建 | 开源（MIT），本体免费；模型调用需 DeepSeek 或其他兼容 Provider 的 API Key，按 API 用量计费 |
| Claude Code | Native Install／Homebrew／WinGet，Linux 另有 apt、dnf、apk | 多数形态需要 Claude 订阅或 Anthropic Console 账号；终端 CLI、VS Code、JetBrains 形态支持接入第三方 provider |
| Codex | 独立安装脚本、npm、Homebrew 等 | 首次运行用 ChatGPT 账号登录（或其他可用登录方式） |
| OpenCode | install 脚本、npm、brew 等 | 开源，本体免费；自带任意 LLM provider 的 API key，或用官方筛选模型的 OpenCode Zen（注册充值） |

四家都有明确获取路径，但「本体开源」不等于「完整使用零成本」。Claude Code 与 Codex 主要走厂商账号或相应 API／用量方案；DeepSeek Harness 和 OpenCode 的典型路径则是自己配置 Provider Key。具体订阅档位和赠送额度变化频繁，本篇不写未经当日核对的价格数字。

网络可达性、账号区域与企业合规条件不会由产品功能页替你保证。真正选型前，应在团队实际网络、账号和合规边界下分别做最小连通测试；本文不把某个地区或某个时点的访问结果包装成长期结论。

> 💡 **一句话总结**：先区分开源本体、账号资格与模型用量三层成本，再在自己的网络和合规环境中做最小连通测试。

---

## 03 模型绑定：一个模型打天下，还是随便换

上一篇说过，模型决定能力上限。那每个工具让你选模型的自由度有多大？

| 产品 | 模型绑定程度 | 当日核对的依据 |
|---|---|---|
| DeepSeek Harness | 以 DeepSeek 官方 Provider 为默认，Host 内置 38 个可配置 Provider 条目，支持自定义兼容 Provider | 固定版本实测（见 09、10 篇） |
| Claude Code | 主路径是 Claude 模型；终端 CLI、VS Code、JetBrains 支持第三方 provider | 官方 overview 页 |
| Codex | 主路径是 OpenAI 模型，ChatGPT 账号体系 | 官方 CLI 页 |
| OpenCode | 设计上就是多模型：`provider/model` 形式任选，OpenCode Zen 是官方筛选清单 | 官方文档与配置页 |

**关键判断：模型绑定不是缺点，是策略。** Claude Code 和 Codex 把模型和 Harness 一起调优，体验闭环最完整，但你的工作流会沉淀在某一家账号里。Harness 和 OpenCode 把模型当可替换件，代价是你要自己管理 Key、价格和模型差异。

对国内开发者，这里有个现实考量：如果团队的主力模型已经是 DeepSeek（成本、合规、网络都顺），那么围绕 DeepSeek 模型设计的 Harness 天然少一层适配。

这次写 DeepSeek Harness 教程，我没有把订阅制和自带 API Key 当成非此即彼。日常写作与编程更看重开箱即用，教程验收却必须固定 Provider、模型和 usage，所以我给 Harness 单独使用按量 API Key。首轮一共跑了 188 次真实请求，模型分布和 Token 都能从 Session 里复核；对我这种小规模内容项目来说，这比为了实验再增加一套团队订阅更容易控制变量，也更方便把实际消耗说清楚。

> 💡 **一句话总结**：打包产品用模型绑定换体验闭环，开放产品用自由度换运维成本，选哪边取决于你的主力模型和账号策略。

---

## 04 架构开放性：能改多少，决定了能走多远

这是 DeepSeek Harness 和其他三家差异最大的地方，也是这套教程存在的理由。

| 维度 | DeepSeek Harness | Claude Code | Codex | OpenCode |
|---|---|---|---|---|
| 源码 | 开源（MIT） | 官方文档未把核心产品列为开源组件 | CLI、SDK、App Server 开源；IDE 扩展与云端产品不开源 | 开源 |
| 扩展机制 | 一切皆插件（Cordis 插件树），工具、Provider、Preset、UI 卡片、Conversation Node 都可替换 | hooks、MCP、Agent SDK | hooks、MCP、插件 | 插件、MCP、自定义 agent |
| 拦截点 | 规范拦截点上的类型化 Decision（`tools/pre-execute` 等） | hooks（当日官方列出 33 个事件） | hooks（当日官方列出 11 个事件） | 插件内 hooks |
| 自定义运行模式 | Agent Preset 整体定义工具面、persona、模式 | 可定义 subagent 与扩展行为；本轮未核到等价 Preset | 支持 modes、profiles、plugins；本轮未核到等价 Preset | 自定义 agent（模型 + 提示词 + 工具开关） |
| 嵌入自己系统 | Python SDK、JSON-RPC、Headless、可嵌入运行时 | Agent SDK | `codex exec`、SDK | `opencode run`、serve |

**类比：装修。** 四家都允许装修，但开放的楼层不同：Claude Code 主要给成品上的 hooks、MCP 和 Agent SDK；Codex 还开放 CLI、SDK、App Server、plugins 与 hooks；OpenCode 开放多模型 Agent 本体；DeepSeek Harness 则把插件树、配置分层、Capability Seam 和 Preset 直接放在运行时架构里。

这里要说句公道话：**开放性不是免费的。** 官方 README 明确写了 Harness 处于 Developer Preview，「未来将出现破坏兼容性的变更」。你得到的是改造能力，付出的是跟踪变化、自己验收的成本。只想安静写业务代码的人，精装房是合理选择，不丢人。

> 💡 **一句话总结**：四家的开放边界不同；Harness 的特点是把运行时层级本身作为主要改造对象，而更深的改造也意味着更重的维护责任。

---

## 05 权限与安全：谁能管住模型的手

模型决定不了安全，Harness 才管得住。四个产品在这层的思路差异很能说明问题：

| 维度 | DeepSeek Harness | Claude Code | Codex | OpenCode |
|---|---|---|---|---|
| 权限模型 | 权限预设 + 四种审批结果，事件持久化可审计 | 权限模式 + hooks 可拦截 | `/permissions` 沙箱与审批，hooks 可拦截可改写 | `permission` 配置可对 edit、bash 设 ask |
| 沙箱 | 三种沙箱模式，fail-closed：无可用沙箱就拒绝执行 | 权限审批为主 | 本地沙箱与可写根配置 | 审批配置为主 |
| 拦截语义 | 类型化 Decision，最严格合并，串行可预测 | hook 退出码与 JSON 决策 | hook `permissionDecision` + `updatedInput` 改写 | 插件拦截 |
| 审计 | Session Event 全量追加，工具调用前后都有事件 | hooks 可记录 | hooks 可记录 | 视插件而定 |

**Harness 的安全设计哲学是 fail-closed：没有答案就是拒绝。** 沙箱后端不可用时不降级放行，而是直接拒绝执行并报 `SANDBOX_UNAVAILABLE`。这条对把 Agent 接入内部系统的人很重要：宁可任务失败，不可边界失守。

另外注意一个容易混淆的点：hooks 是「用户自定义的拦截脚本」，Harness 的审批与沙箱是「产品内置的安全边界」。前者是扩展，后者是底线。不同产品都提供拦截或权限扩展；Harness 在固定版中把规范拦截点做成带类型的 Decision 协议。

> 💡 **一句话总结**：日常任务先看各自权限与审批是否覆盖需求；接入内部系统时，再重点比较类型化拦截、默认拒绝、审计证据和部署边界。

---

## 06 决策路径：什么人选什么

铺垫完了，直接给路径。按你的真实处境对号入座：

```text
你只想马上有个最强的编码助手，不想管原理
├─ 主力模型是 Claude → Claude Code
└─ 主力模型是 GPT 系 → Codex

你想自由换模型、自带 Key、工具开源可改
└─ OpenCode

你是以下任意一种人：
├─ 想真正搞懂 Agent 怎么工作（而不只是用）
├─ 团队主力模型是 DeepSeek，要深度定制工作流
├─ 要把 Agent 能力嵌入自己的产品或内部系统
├─ 要开发自己的工具、Provider、运行模式
└─ DeepSeek Harness
```

再诚实一点，说说不适合选 Harness 的情况：

| 你的情况 | 建议 |
|---|---|
| 只是想找个编码助手提效 | 三家成品都更省事，Harness 的教程价值大于直接使用价值 |
| 团队没有 Node.js／TypeScript 维护能力 | Developer Preview 的破坏变更会消耗你 |
| 需要官方 SLA 和企业支持 | 成熟商业产品更合适 |
| 追求「装上就永远不变」 | Developer Preview 明确警告会有破坏兼容性变更 |

我在这套教程里安排过一次很直观的层级对照：修复 `calculator.js` 里一个减号，本教程用固定版 `npx` 加 Standard 模式大约 9 秒就完成；为了研究 Harness 本身，我们又 checkout 源码、冻结 246 个 workspace 和 937 个 package，还处理了 React 18／19 类型污染。后者不是更高级的修 Bug 方法，而是研究基础设施必须支付的成本。如果目标只是改业务代码却选择源码级 Harness 开发，绝大部分时间都会花在维护工具，而不是交付任务。

反过来，如果你读这套教程读到这一篇还想继续，大概率你属于最后一类：你关心的不只是「用 Agent」，而是「Agent 这层基础设施本身」。这正是本系列后 82 篇要讲的。

> 💡 **一句话总结**：要成品选 Claude Code 或 Codex，要多模型开源成品选 OpenCode，要理解、定制、嵌入 Agent 基础设施选 DeepSeek Harness。

---

## 07 常见误判：四个容易踩的坑

对比类文章最容易制造错觉，这里提前拆掉四个：

| 误判 | 真相 |
|---|---|
| 「哪个工具跑分高用哪个」 | 跑分大多是模型分数，不是 Agent 体验分数；同一模型在不同 Harness 里表现不同（02 篇的公式） |
| 「开源的一定便宜」 | 壳免费，模型调用照样计费；自维护的时间也是成本 |
| 「功能列表长的赢」 | 大量常见任务都能完成；真正差异在边界情况、默认行为和可改造性 |
| 「选定一个就不用换」 | 这个领域按月在变；本篇所有竞品事实都标注了核对日期，过期要重查 |

特别强调第一个：社交媒体里的「X 吊打 Y」，很多没有固定模型、任务和提示词。**控制变量应该是：同模型、同任务、同验收标准，只换 Harness。** 这恰好是开放运行时适合做的实验--可以固定模型只换配置。

> 💡 **一句话总结**：抛开模型谈工具强弱是耍流氓，抛开任务谈体验也是；对比的唯一公平方式是控制变量。

---

## 08 本篇事实的核对记录（live-external）

本篇是 `live-external` 标签文章，竞品事实按日过期。2026-08-31（北京时间）核对结论：

| 核对项 | 当日结果 |
|---|---|
| DeepSeek Harness 最新 Release | `dsh-v0.1.2-alpha.2`（prerelease，2026-08-30 发布，npm 上仅以 `alpha` dist-tag 提供）；npm `latest` 仍为 `0.1.1-rc.2`，与本系列固定版本一致 |
| Claude Code 官方文档 | overview 与 hooks 参考页均可达；安装口径为 Native Install／Homebrew／WinGet 等；hooks 事件表当日共 33 项 |
| Codex 官方文档 | CLI、hooks 与 Open Source 页均可达；hooks 事件当日共 11 项；CLI、SDK、App Server 开源，IDE 扩展与云端产品不开源 |
| OpenCode 官方文档 | 文档首页与配置页均可达；开源、多 provider、OpenCode Zen 口径不变 |

完整核对表（URL、HTTP 状态、逐条事实）在 `references/deepseek-harness/COMPARE-BASELINE.md`。没有当日核对到的内容——比如各家的具体价格数字、订阅档位差异——本篇一律不写，宁可留白也不凭记忆。

> 💡 **一句话总结**：竞品事实的保质期以天计，本篇只写当日核到的，没核到的一律标注待复核。

---

## 09 如果你已经在了别家：迁移的成本真相

最后一种常见处境：你已经是 Claude Code 或 Codex 的老用户，手里攒了一堆 CLAUDE.md／AGENTS.md、hooks 脚本和肌肉记忆，迁移到 Harness 要付出什么？

先说结论：**迁移的主要成本不在「学会新命令」，而在「搬你的工作环境」。** 具体是四类资产：

1. **项目指令**：CLAUDE.md／AGENTS.md 这类文件。Harness 会在 agent 工作目录下发现 AGENTS.md 风格的指令文件，概念可直接对应。
2. **hooks 脚本**：Harness 官方提供 `dsh-hooks-claude-code` 和 `dsh-hooks-codex` 两个桥接插件，能在 Harness 的规范拦截点上直接运行你现有 hooks 配置的受支持子集。
3. **权限与审批习惯**：从「权限模式 + hook 拦截」迁移到「权限预设 + 类型化审批」。
4. **肌肉记忆**：斜杠命令、交互习惯，这部分只能重新练。

桥接插件能搬第 2 类资产，但它是「兼容路径」不是「完整复刻」：Claude Code 33 个 hook 事件里桥接支持 7 个，Codex 11 个里支持 5 个，且部分决策字段（如 `updatedInput`）会被记录但不生效。官方自己的建议是：定制行为最终应该用原生 Cordis 插件重写。

这个项目最近把原来偏 Claude Code 的协作入口收敛成 `AGENTS.md` 单一内容源，再让 `CLAUDE.md` 只保留 `@AGENTS.md` 兼容入口。文字规则本身迁得很快，真正需要逐条判断的是哪些属于全局红线、哪些应分流到 `docs/` 和 `ROADMAP.md`，以及不同 Agent 对 Skill、审批和发布动作的语义是否一致。最后我没有把 hooks 和工具配置强行做成一份通用文件，因为它们不是同一种协议；能共享的指令共享，运行时扩展继续按各自机制维护。

> 💡 **一句话总结**：迁移成本集中在指令、hooks、权限习惯三类资产，桥接插件能救急，长期要用原生插件重写。

---

## 小结

四家产品放到「Agent = Model + Harness」的尺子下，答案其实很简单：

- **Claude Code 和 Codex**是模型厂商的打包成品，模型绑深、体验闭环、开箱即用，适合「只要最强助手」的人。
- **OpenCode**是开源的多模型成品，模型自由换，适合「自带 Key、不想被账号绑定」的人。
- **DeepSeek Harness**是开源的 Agent 基础设施，把 Harness 本身做成可改造对象，适合想理解原理、深度定制、嵌入自己系统的人——代价是接受 Developer Preview 的变化节奏。

你现在应该能不看任何评测文，自己判断一个新出的编码工具属于哪一层、适不适合你。如果你最终的选择是 Harness，下一篇咱们回到概念，把模型、Agent、Harness、IDE 这条链彻底理清。

---

下一篇：[04 · 模型、Agent、Harness、IDE 之间是什么关系](./04-model-agent-harness-ide.md)。如果你在 Claude Code 或 Codex 已经攒了一堆配置想搬家，可以直接跳去 [83 · 从 Claude Code／Codex 迁移到 Harness](./83-migrate-from-claude-code-codex.md)。
