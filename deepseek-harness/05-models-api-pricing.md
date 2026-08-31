---
seoTitle: "DeepSeek 模型目录、API Key、价格与 Harness 调用方式"
description: "集中核对 DeepSeek V4 三个当前 API 模型、1M 上下文、峰谷价格、API Key 安全边界与 Harness 固定版本调用方式，并区分官方服务上限、Harness 默认值和真实账号待验证项。"
lastVerified: "2026-08-30"
author: stormzhang
dshVersion: "0.1.1-rc.2"
verificationLevel: "V2"
verificationStatus: "online-facts-rechecked-2026-08-30-real-provider-passed-owner-experience-confirmed"
stability: "live-external"
officialSources:
  - "https://api-docs.deepseek.com/zh-cn/quick_start/pricing/"
  - "https://api-docs.deepseek.com/zh-cn/"
  - "https://api-docs.deepseek.com/api/list-models/"
  - "https://deepseek-harness.github.io/deepseek-harness/guide/providers"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/packages/llm/llm-deepseek/README.zh.md"
related:
  - "./09-deepseek-provider.md"
  - "./10-custom-provider.md"
evidence:
  - "references/deepseek-harness/ARTICLE-SOURCE-MAP.md：第 05 篇"
  - "references/deepseek-harness/RUNTIME-BASELINE.md：Provider 与模型目录"
  - "2026-08-30 北京时间发布前重查 DeepSeek 官方模型与价格页"
---

# 05 · DeepSeek 模型目录、API Key、价格与调用方式

> 📚 **系列导航**：上一篇 [04 · 模型、Agent、Harness、IDE 的关系](./04-model-agent-harness-ide.md) 建立了分层地图。这一篇只处理变化最快的一层：当前有哪些 DeepSeek API 模型、怎么计费，以及 Harness 怎样接上它们。

> [!WARNING] 实时外部事实
> 本文模型、规格、价格与峰谷时段已于 2026-08-30 在线重查 DeepSeek 官方页面，与文中一致，并已用真实 Key 完成 `/models` 与最小 Agent Turn 验证；价格属高频变化信息，以官方价格页当日口径为准。

模型和价格是整套教程里最容易过期的内容。

旧文章还在讲 `deepseek-chat`、`deepseek-reasoner`，当前官方主线已经变成 DeepSeek V4；Harness 固定版又额外带着自己的模型目录和请求默认值。**如果把「官方今天提供什么」和「Harness 这个版本默认列出什么」混在一起，教程当天就可能写错。**

先给结论：截至 2026-08-30，DeepSeek 官方价格页列出三个模型：

- `deepseek-v4-flash`
- `deepseek-v4-pro`
- `deepseek-v4-flash-vision-exp`

Harness `0.1.1-rc.2` 的 `deepseek-official` Provider 默认目录也正好公布这三个模型，但这只是当前对齐，不能当作永远不变的约定。

**看完这一篇，你会拿到：**

- 当前三个 DeepSeek V4 API 模型的能力与适用边界
- 一张人民币峰谷价格表，能自己估算一次 Agent 任务的 Token 成本
- 分清官方 API 上限、Harness 默认请求上限和模型选择器目录
- 知道 API Key 在 Harness 里怎样保存，避免把密钥写进仓库或截图
- 两个安全检查动作：确认环境变量状态，并查询账号实际可用模型

---

## 01 两份模型目录：官方服务和 Harness 固定版本

先看今天的官方 API 价格页：

| 模型 ID | 官方模型版本 | 输入模态 | 定位提醒 |
|---|---|---|---|
| `deepseek-v4-flash` | DeepSeek-V4-Flash-0731 | 文本 | V4 快速模型，支持思考与非思考模式 |
| `deepseek-v4-pro` | DeepSeek-V4-Pro-0813 | 文本 | V4 Pro，支持思考与非思考模式 |
| `deepseek-v4-flash-vision-exp` | DeepSeek-V4-Flash-Vision-Exp | 文本、图片 | 视觉实验模型，名字里的 `exp` 就是实验性提醒 |

三个模型在官方页面当前都标注：

- 上下文长度为 **1M Token**。
- 最大输出为 **384K Token**。
- 支持 JSON Output、Tool Calls、Responses API 与 Anthropic API。
- 默认启用思考模式，也能切换到非思考模式。
- 前缀续写仍是 Beta；FIM 只在前两个文本模型的非思考模式下支持，视觉实验模型不支持 FIM。

再看 Harness 固定版本。`@deepseek-ai/dsh-llm-deepseek` 注册的 Provider ID 是 `deepseek-official`，省略自定义 `models` 时，同样公布：

```text
deepseek-v4-flash
deepseek-v4-pro
deepseek-v4-flash-vision-exp
```

G02 的真实 Host 目录查询也得到了这三个模型。这能证明 **Harness 固定版本的目录**，但不能代替真实账号调用：账号是否有权限、模型是否临时维护、API 是否返回 401／429，都要用真实 Key 才知道。

| 你看到的地方 | 它能证明什么 | 它不能证明什么 |
|---|---|---|
| 官方价格页 | 官方当前公开模型、规格与标价 | 你的账号一定能成功调用 |
| API 文档的 `/models` 示例 | 接口结构和示例 ID | 你的账号实时返回完全相同 |
| Harness 模型选择器 | 本地 Provider 公布的建议目录 | 远端模型可用、余额充足 |
| 真实 `GET /models` | 当前 Key 能查询到的模型目录 | 某个模型请求一定成功 |
| 真实 Agent Turn | Provider、模型和 Harness 链路实际可用 | 后续每次都不会限速或失败 |

这套教程第一次接入真实 DeepSeek API 是 2026 年 8 月 29 日 01:22（北京时间）。我要求 Key 不写进设置文件，只用环境变量注入；验收先请求 `/models`，HTTP 200，随后才跑最小 Agent Turn。充值金额和账号余额没有进入验收记录，因为它们属于账号隐私，也不能证明单次任务成本；我宁愿把可核对的模型 ID、usage 和请求结果留下，也不为文章补一个无法复核的金额。

> 💡 **一句话总结**：官方目录、Harness 目录和真实账号目录是三份不同证据；当前名字一致，不代表可以互相替代。

---

## 02 三个模型怎么选

官方页面给了规格和价格，但没有替你定义所有任务的绝对排名。没有真实同任务测试之前，教程不会编造「Flash 一定快多少」「Pro 一定强多少」这类数字。

现阶段可以按能力边界做保守选择：

| 任务 | 建议起点 | 理由 |
|---|---|---|
| 仓库概览、简单修改、批量低风险任务 | `deepseek-v4-flash` | 单价更低，适合作为首个连通性与日常任务基线 |
| 复杂排错、长链路规划、高难度代码推理 | `deepseek-v4-pro` | 官方 Pro 路线，成本更高，值得在复杂任务上对照 |
| 截图、界面或图片附件理解 | `deepseek-v4-flash-vision-exp` | 当前三个模型中明确声明图片输入的实验模型 |
| 只想确认 Harness 是否装好 | Mock Provider | 不消耗真实 Token，但不能评价模型质量 |

**类比：先选车型，再看驾驶系统。** Flash、Pro、Vision 是模型车型；Standard、PTC、Minimal、Creator 是 Harness 的驾驶与工具配置。你可以用 Flash 跑 Standard，也可以用 Pro 跑 PTC，它们不是一一绑定的套餐。

这里还有两个边界：

1. 视觉模型虽然出现在固定版默认目录中，但 Files API、图片大小和真实请求仍未完成本教程实测，不能只凭选择器有名字就标记「视觉功能已验证」。
2. 模型 ID 会原样传给 DeepSeek API。Harness 目录是建议列表，不是硬白名单；手工传入未列出的 ID 不会因此变成受官方支持的模型。

> 💡 **一句话总结**：先按文本／视觉和任务复杂度选模型，再按工作方式选 Preset；模型与 Harness 模式是两个独立旋钮。

---

## 03 2026-08-30 官方人民币价格

DeepSeek 当前采用峰谷价格，单位都是**每 100 万 Token**。北京时间周一至周五的两个高峰时段为：

- 09:00～12:00
- 14:00～18:00

其余时间按空闲时段价格计费。官方当前价格如下：

| 计费项 | Flash／Vision 空闲 | Flash／Vision 高峰 | Pro 空闲 | Pro 高峰 |
|---|---:|---:|---:|---:|
| 输入，缓存命中 | ¥0.05 | ¥0.10 | ¥0.15 | ¥0.30 |
| 输入，缓存未命中 | ¥1.50 | ¥3.00 | ¥4.50 | ¥9.00 |
| 输出 | ¥4.50 | ¥9.00 | ¥13.50 | ¥27.00 |
| 账号并发上限 | 2500 | 2500 | 500 | 500 |

视觉模型的图片会按尺寸换算成输入 Token，再与文本 Token 一起计费。并发上限是账号级，不是每个 API Key 单独一份。

举个能落地的成本例子。假设一次 Agent 任务消耗：

- 10 万输入 Token，全部缓存未命中。
- 1 万输出 Token。

那么：

| 模型与时段 | 输入成本 | 输出成本 | 合计 |
|---|---:|---:|---:|
| Flash 空闲 | 0.1 × ¥1.50 = ¥0.15 | 0.01 × ¥4.50 = ¥0.045 | **¥0.195** |
| Flash 高峰 | 0.1 × ¥3.00 = ¥0.30 | 0.01 × ¥9.00 = ¥0.09 | **¥0.39** |
| Pro 空闲 | 0.1 × ¥4.50 = ¥0.45 | 0.01 × ¥13.50 = ¥0.135 | **¥0.585** |
| Pro 高峰 | 0.1 × ¥9.00 = ¥0.90 | 0.01 × ¥27.00 = ¥0.27 | **¥1.17** |

这只是教学估算。Agent 会多次请求模型，系统提示词、工具 schema、消息历史和工具结果都会占输入 Token；上下文缓存是否命中也不能靠主观猜，要看 API usage 中的 `prompt_cache_hit_tokens` 与 `prompt_cache_miss_tokens`。

第一轮真实 Key 验收共发起 188 次 LLM 请求，其中 Flash 187 次、Pro 1 次；新鲜输入约 27.6 万 Token、缓存读取约 140 万、输出约 5.6 万。最小 Flash Turn 单次是输入 7618、输出 25、缓存读取 0、推理 19，后续多 Step 任务能看到 `cacheReadTokens` 明显增加。我没有记录账单绝对金额，所以这里不写「大约几分钱」这种看似精确的估算；真实结论只到 Token 和缓存命中这一层。

> [!TIP] 价格维护规则
> 本表只代表 2026-08-30 官方页面。每次发布或复核本文，都必须重新打开官方价格页；不要从本文复制数字到其他 10 篇文章里，避免一处涨价、全站过期。

> 💡 **一句话总结**：费用由多次请求的输入、输出和缓存状态共同决定；Agent 成本不能只拿一条用户提示词的字数估算。

---

## 04 API Key 在 Harness 里怎么走

首次使用最推荐的路径是 Web UI：

```text
设置
→ 模型
→ DeepSeek 卡片
→ 输入 API Key
→ 保存
```

官方当前文档说明，Key 是**只写**的：保存后，页面只会收到脱敏描述符，不会把明文密钥再读回浏览器。密钥存储在：

```text
$DSH_HOME/.credentials.yaml
```

`settings` 中只保存凭据引用。保存成功后，下一次模型请求立即使用新配置，不需要重启服务器。

Harness 固定版的 DeepSeek 适配器默认引用环境变量名 `DEEPSEEK_API_KEY`。每次请求时，它会先通过凭据服务解析，再回退到受信环境层。没有任何可用密钥时，Provider 仍然注册、模型目录仍可浏览，但真正请求会以 `MISSING_CREDENTIAL` 失败。

| 做法 | 是否推荐 | 原因 |
|---|---|---|
| 在 Web UI 的凭据字段保存 | 推荐 | 明文不回传页面，配置与凭据引用分离 |
| 临时使用 `DEEPSEEK_API_KEY` 环境变量 | 可用于 Headless／临时验证 | 不写入项目文件，但要注意 Shell 历史和进程环境 |
| 把 Key 写进 Markdown、代码或 Git | 禁止 | 会进入仓库、构建产物或分享记录 |
| 把完整 Key 截图发给别人排错 | 禁止 | 脱敏前已经泄露，删除截图也不等于密钥失效 |
| 在日志里打印 Key 验证是否配置 | 禁止 | 配置检查不需要输出秘密值 |

如果怀疑环境变量没有生效，用下面的安全命令，只检查「有没有」，不打印内容：

```bash
node -e 'console.log(process.env.DEEPSEEK_API_KEY ? "DEEPSEEK_API_KEY=CONFIGURED" : "DEEPSEEK_API_KEY=MISSING")'
```

预期输出二选一：

```text
DEEPSEEK_API_KEY=CONFIGURED
```

```text
DEEPSEEK_API_KEY=MISSING
```

> 💡 **一句话总结**：检查密钥状态，不检查密钥内容；Key 应进入 Harness 凭据层或临时环境变量，绝不能进入代码和教程。

---

## 05 官方 API 上限不等于 Harness 默认请求值

这是本篇最容易写错的细节。

DeepSeek 官方今天标注的模型规格是：**1M 上下文、最大 384K 输出**。Harness `0.1.1-rc.2` 适配器的默认值则是：

| 配置 | Harness 固定版默认值 | 含义 |
|---|---:|---|
| `defaultContextWindow` | 1,000,000 | 未给具体模型容量时的上下文回退值 |
| `maxTokens` | 256,000 | Harness 发起对话请求时的默认输出上限 |
| `reasoningEffort` | `high` | 默认推理强度 |
| `thinking` | `enabled` | 默认启用思考模式 |
| `streamIdleTimeoutMs` | 300,000 ms | 流式读取五分钟无活动超时 |

所以不能写成「Harness 默认输出 384K」。**384K 是官方模型最大输出，256K 是这个 Harness 适配器当前默认请求上限。** 显式请求值、Agent 配置或模型目录中的 `maxTokens` 还可以继续覆盖适配器默认值。

固定版 Harness 对推理强度公开 `off`、`low`、`high`、`max`：

- `off` 会发送 `thinking.type: disabled`，并省略 `reasoning_effort`。
- `low`、`high`、`max` 会启用思考，并发送同名推理强度。
- 不支持的值会在网络请求前失败，不会默默降级。

DeepSeek 官方 OpenAI 格式 Base URL 是：

```text
https://api.deepseek.com
```

Anthropic 格式 Base URL 是：

```text
https://api.deepseek.com/anthropic
```

Harness 的 `deepseek-official` 适配器走 DeepSeek Chat Completions 协议，并用 `fetch + SSE` 转换为内部流式事件。用户正常使用 Web UI 时，不需要自己手写 curl；这些协议细节主要用于理解错误和开发 Provider。

> 💡 **一句话总结**：提供方能力上限、Harness 适配器默认值和单次 Agent 请求值是三层配置，看到数字先问它属于哪一层。

---

## 06 动手：查询真实账号模型目录

这一步需要你已经设置 `DEEPSEEK_API_KEY`，命令不会打印 Key 本身：

```bash
curl --silent --show-error https://api.deepseek.com/models \
  -H "Authorization: Bearer ${DEEPSEEK_API_KEY}"
```

成功时会返回类似结构：

```json
{
  "object": "list",
  "data": [
    {
      "id": "deepseek-v4-flash",
      "object": "model",
      "owned_by": "deepseek"
    }
  ]
}
```

这里故意不把三个模型全部写进「预期输出」。`GET /models` 的价值就是查询**当前账号的真实目录**，结果应与调用时的官方页面和账号权限核对，而不是为了符合教程样例强行假设。

常见结果怎么判断：

| 结果 | 含义 | 下一步 |
|---|---|---|
| HTTP 200 + `data` 列表 | Key 可用于查询模型 | 记录实际模型 ID，再做最小 Agent Turn |
| HTTP 401 | Key 缺失、错误或失效 | 回到平台创建／轮换 Key，检查环境变量 |
| HTTP 429 | 余额、配额或并发相关限制 | 查看响应错误和账号状态，不要盲目重试 |
| DNS／TLS／连接失败 | 网络或代理问题 | 先验证能否访问 `api.deepseek.com` |

本教程已于 2026 年 8 月 29 日使用环境变量注入的真实 Key 执行这条命令，HTTP 200，返回三个模型 ID；Key 本体没有进入日志、文档或 Git。验收保存了以下证据，但没有保存密钥：

```text
验证时间、Harness 版本、模型 ID 列表、HTTP 状态、最小 Agent Turn 结果、usage、错误码
```

同一次验收中，真实 `/models` 恰好返回 `deepseek-v4-flash`、`deepseek-v4-pro` 和 `deepseek-v4-flash-vision-exp` 三个 ID。我安排同一条固定回复任务各跑一次 Flash 和 Pro：Flash 输入 7618、输出 25，耗时约 0.7 秒；Pro 输入 7596、输出 24，耗时约 1.5 秒，两者都一次成功。这个极小任务看不出质量差异，因此我继续用 Flash 做教程默认基线，把 Pro 留给更复杂任务，而不是仅凭名字给它判定更强。

> 💡 **一句话总结**：文档目录告诉你官方公开什么，`GET /models` 才告诉你当前 Key 实际看见什么；最后还要跑一次 Agent Turn 才算链路闭合。

---

## 小结

这一篇把最容易变化的事实集中在了一处：

1. 2026-08-30 官方主线是 `deepseek-v4-flash`、`deepseek-v4-pro` 与 `deepseek-v4-flash-vision-exp`。
2. 三者当前都标注 1M 上下文、最大 384K 输出；视觉模型是实验项。
3. 价格按百万 Token 和峰谷时段计费，输入还要区分缓存命中与未命中。
4. Harness 固定版 Provider ID 是 `deepseek-official`，默认目录与官方当前三模型对齐。
5. 官方 384K 输出上限不等于 Harness 默认 `maxTokens: 256000`。
6. Key 只能进入凭据层或受控环境变量，不能进入 Git、文档、截图和日志。
7. 本文已完成真实 Key 的 `/models`、最小 Agent Turn、usage 与 Flash／Pro 对照；视觉模型请求仍待补。

---

首发路线下一篇：[06 · 环境准备](./06-requirements.md)。真正配置 DeepSeek Provider 的完整界面与排错流程，会在 [09 · 配置 DeepSeek 官方 Provider](./09-deepseek-provider.md) 继续展开。
