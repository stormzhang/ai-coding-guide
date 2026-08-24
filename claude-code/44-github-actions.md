---
seoTitle: "Claude Code GitHub Actions 集成教程"
description: "在 GitHub Actions 和 PR 中调用 Claude Code 的安装、认证、工作流配置、触发方式与安全注意事项，适合仓库自动化场景"
published: "2026-06-12"
lastVerified: "2026-06-20"
author: stormzhang
officialSources:
  - https://code.claude.com/docs/zh-CN/github-actions
related:
  - ./41-parallel-tasks.md
  - ./47-voice.md
---
# 44 · GitHub Actions：在 PR 里 @ 一下，让 Claude 自己干活

> 📚 **系列导航**：上一篇 [43 Git 工作流](43-git-workflow.md) 教你怎么让 Claude 在本地帮你管分支、写提交、开 PR——那都是**你坐在电脑前、它在你身边干**。这一篇把它送上云端：**配好之后，你在 GitHub 的 issue 或 PR 里打一句 `@claude`，它就自己跑起来分析代码、改东西、开 PR，全程不用你开电脑**。这就是 Claude Code GitHub Actions（GitHub 自动化工作流平台）。

设想这么一个半夜十一点多的场景：你已经躺下了，团队群里弹出一条消息。

一个同事：「老 X 那个 PR 你看了吗？卡了一天了，明天要上线。」
你：「我睡了，明天一早看。」
他：「……明天一早就来不及了啊。」

这种时候，要是早给那个仓库配上了 Claude Code GitHub Actions，就能省下无数个这样的夜晚：**那个 PR 还没等你醒，评论区里已经躺着一份 Claude 自动跑出来的 review——逐行标出了一个空指针风险和两处没处理的边界**。同事直接照着改完合了。

说白了，前面四十多篇我们聊的 Claude Code，**都得你人在、终端开着、你盯着它干**。GitHub Actions 这一篇要解决的是另一件事：**让它脱离你的电脑，住进 GitHub 的服务器里，靠一句 `@claude` 就能被喊起来干活**。你出差、你睡觉、你在开会，它照样能接 issue、审 PR、改 bug。

**看完这一篇，你会拿到：**

- 一句话搞懂 Claude Code GitHub Actions 是什么、它和你本地用的 Claude Code 是什么关系
- `@claude` 提及触发到底怎么回事——你在哪打、它怎么知道该不该响应
- 最小可跑的 workflow YAML 长什么样、每一行在干什么，照着抄就能用
- 三个最实用的用例：自动 code review、按 issue 自动改、定时任务
- API key / 密钥怎么安全配进 GitHub，绝不能踩的那条红线
- 一个能照着走、给了预期结果的实战：5 分钟把它装进你自己的仓库并验证

---

## 01 先搞懂：它是「住进 GitHub 的 Claude Code」

先给结论：**GitHub Actions 版的 Claude Code，就是把你本地那个 Claude Code，搬到 GitHub 的服务器上跑——触发它的不再是你敲终端，而是仓库里发生的事件（有人开 PR、有人评论、定时到点）。**

你回想前面四十多篇，Claude Code 一直是这么个用法：你在自己机器上开个终端，敲 `claude`，然后跟它对话、看它改文件。**它干活的前提是「你在场」**——你得开着电脑、盯着它、随时批准它的操作。

GitHub Actions 把这个前提拿掉了。它的官方定义很直白：

> Claude Code GitHub Actions 为您的 GitHub 工作流带来了 AI 驱动的自动化。只需在任何 PR 或 issue 中简单地提及 `@claude`，Claude 就可以分析您的代码、创建拉取请求、实现功能和修复错误——所有这些都遵循您项目的标准。

**类比：给团队招了个不用睡觉的夜班同事。** 你们组白天有人盯着代码，下班了、半夜了就没人了。现在你招了个夜班的——他不打卡、不需要工位、不要工资（只按用量付费），**只要群里有人 @ 他一句「帮我看下这个 PR」「把这个 bug 修了」，他就接活、干完把结果贴回来**。你睡你的，他干他的。GitHub Actions 版的 Claude 就是这么个夜班同事，只不过他住在 GitHub 的机房里。

这里有个关键点别搞混：**它不是另一个新产品，底层就是 Claude Code。** 官方说得明白——它建立在 Claude Agent SDK 之上（这个第 45 篇会讲），而且它干活时**照样读你仓库根目录的 `CLAUDE.md`**（第 18 篇讲过那份「项目说明书」）。也就是说，你之前给项目写的规矩、约定、风格指南，这个夜班同事**全盘照收**，跟你本地用的是同一套标准。

落到真实场景，配好之后你能干这些：

- **PR 开出来了，没人有空 review**——让它自动审一遍，逐行标出问题
- **issue 描述清楚了，但你没空写代码**——评论一句 `@claude 把这个实现了`，它直接开个 PR 把功能实现了
- **CI 挂了，半夜没人管**——让它分析报错、尝试修复

> 💡 一句话总结：GitHub Actions 版 Claude Code 就是**搬进 GitHub 服务器、靠仓库事件触发的 Claude Code**；它不用你在场，照样读你的 `CLAUDE.md`、守你的项目标准，像个不用睡觉的夜班同事。

---

## 02 `@claude` 提及：你怎么喊它，它怎么知道该不该来

最核心的用法——**在 issue 或 PR 的评论里 @ 它一句，它就动**。这一节把这件事拆透：你在哪打、打什么、它凭什么响应。

**类比：在群里 @ 一个值班同事派活。** 你们团队群里，平时聊天刷屏，但只要你 @ 了那个值班的，加一句具体要求，他就知道「这条是叫我的」，然后接活去办。`@claude` 一模一样——**它平时不插话，你不 @ 它，它当没看见；你一 @ 它，它就把这条评论当成派给自己的任务**。

具体在哪打？官方给的几个地方，都是你日常会用的：

- **PR 的评论区**（issue comment）
- **PR 代码行上的 review 评论**（pull request review comment）
- **issue 的正文或评论**

打什么内容？就跟你给真人同事派活一样，**说人话、说具体**。比如这几句：

```text
@claude 按这个 issue 的描述把功能实现了
@claude 这个接口的用户认证该怎么做
@claude 把用户面板组件里那个 TypeError 修了
```

分别是「按这个 issue 的描述把功能实现了」「这个接口的用户认证该怎么做」「把用户面板组件里那个 TypeError 修了」。**它会自动分析上下文——读 issue、读相关代码、读你的 `CLAUDE.md`——然后做出对应的响应**：能直接改的就开 PR，是提问就回答。

这里接上第 15 篇那条铁律——**指令越具体，它越不跑偏**。你在群里跟真人说「这个 PR 看一下」和「这个 PR 重点查一下并发安全和 SQL 注入」，得到的结果天差地别。`@claude` 也一样，含糊一句「看看」它只能泛泛扫，点名「查 SQL 注入漏洞」它就往那个方向使劲。

有个新手最容易踩的坑，官方专门在故障排除里点了名：

> 确认评论包含 `@claude`（不是 `/claude`）。

很常见的一幕：配好之后兴冲冲在 PR 里打了句 `/claude review this`，等了五分钟没动静，以为是密钥配错了，翻来覆去查了半天。**最后发现是把 `@` 打成了斜杠**——`/claude` 是你本地终端里的斜杠命令（第 36 篇那套），在 GitHub 评论里它谁也不是。**云端触发认的是 `@claude`（@ 符号），不是斜杠。** 记死这一条，能帮你省下半小时排查。

| | 本地终端（前 43 篇） | GitHub Actions（这一篇） |
|---|---|---|
| **谁来触发** | 你敲 `claude` 启动、对话 | 仓库事件（评论、开 PR、定时） |
| **怎么喊它干活** | 直接说话、`/` 斜杠命令 | 评论里 `@claude`（@ 符号！） |
| **你要在场吗** | 要，全程盯着 | 不用，它在云端自己跑 |
| **跑在哪** | 你自己的机器 | GitHub 托管的 runner |
| **批准谁来点** | 你逐步批准 | 按 workflow 预设的权限，无人值守 |

> 💡 一句话总结：在 issue / PR 评论里 **`@claude` + 一句具体要求**就能喊它干活；它自动读上下文和 `CLAUDE.md` 后响应；**千万别打成 `/claude`**——云端认的是 @ 符号，不是斜杠。

---

## 03 装上它：一条命令的「自动安装」最省事

`@claude` 能响应的前提，是你得先把这套东西装进仓库——**装三样：一个 GitHub App（让 Claude 有权限读写你的仓库）、一个密钥（你的 API key）、一个 workflow 文件（告诉 GitHub 什么时候该把 Claude 喊起来）**。

听着麻烦，但官方给了条「一键装」的捷径。**最省事的办法：在你本地的 Claude Code 终端里，跑一条命令。**

```bash
/install-github-app
```

注意——**这条 `/install-github-app` 是在你本地 `claude` 会话里敲的斜杠命令**（对，就是第 36 篇那套斜杠命令，跟上一节说的云端 `@claude` 是两码事，别又搞混了）。它会拉起一个引导流程，官方说它会带你走完：

> 此命令将指导您完成 GitHub 应用和所需密钥的设置。

具体就是帮你把三件事一次办了：装好 GitHub App、把 `ANTHROPIC_API_KEY` 密钥配进仓库、把示例 workflow 文件丢进 `.github/workflows/`。跟着提示点几下就行。

不过有两个**硬门槛**得先满足，官方用 Note 框特意强调过：

> - 您必须是仓库管理员才能安装 GitHub 应用并添加密钥
> - 此快速启动方法仅适用于直接 Claude API 用户。如果您使用 Amazon Bedrock 或 Google Vertex AI，请参阅相应部分。

翻成人话：**第一，你得是这个仓库的管理员**（admin 权限），不然装不了 App、加不了密钥——这是 GitHub 的权限要求，不是 Claude 卡你。**第二，这条捷径只服务「直接用 Claude API」的人**；如果你们公司是走 AWS Bedrock 或 Google Vertex AI 那套云基础设施（第 05 篇提过的第三方模型路子），得走手动配置，这一篇末尾会点到。

那 GitHub App 装上之后要哪些权限？官方列得很清楚，就三项，而且都是读写：

- **Contents**：读写（用来改仓库文件）
- **Issues**：读写（用来响应 issue）
- **Pull requests**：读写（用来创建 PR 和推送更改）

**类比：给夜班同事发一张能进哪些门的工牌。** 这三项权限就是工牌上印的门禁——能进「代码库」改文件、能进「issue 区」回话、能进「PR 区」开单子推代码。**你不给他工牌，他站在门口啥也干不了**；给多了又不安全。这三项是它干活的最小必需，官方默认请求的也正是这三项。

如果 `/install-github-app` 跑失败了（比如你网络不通、或者权限没对上），也能纯手动来：去 [https://github.com/apps/claude](https://github.com/apps/claude) 装 App、去仓库 Settings 里加密钥、从官方仓库的 [examples/claude.yml](https://github.com/anthropics/claude-code-action/blob/main/examples/claude.yml) 复制 workflow 文件。**但能用 `/install-github-app` 就别手动**——两种都走下来，自动那条省心太多，手动那次光找 examples 文件在哪就得翻好几分钟。

> 这套东西需要访问 GitHub 和 Claude 的服务器，国内网络如果不通，装 App、跑 workflow 那几步可能卡住，先开「魔法上网」再试。

> 💡 一句话总结：装它最省事的办法是本地 `claude` 里跑 **`/install-github-app`**，它一条龙帮你装 App、配密钥、放 workflow；前提是你得是**仓库管理员**且走**直接 Claude API**；给的工牌就三项读写权限——Contents、Issues、Pull requests。

---

## 04 workflow YAML：给夜班同事排的那张「值班表」

装好之后，真正决定「什么事发生时该把 Claude 喊起来」的，是那个 workflow 文件——一个躺在 `.github/workflows/` 目录下的 YAML。**这一节我们把最小的那份拆开，逐行看懂。**

**类比：给夜班同事排的值班表。** workflow 文件就是这张表，上面写清楚两件事：**什么情况下他要上班**（触发条件）、**上班了具体干什么**（执行步骤）。没这张表，同事招来了也不知道何时该动、动了干啥。

先看官方给的最小一份「响应 `@claude` 评论」的 workflow：

```yaml
name: Claude Code
on:
  issue_comment:
    types: [created]
  pull_request_review_comment:
    types: [created]
jobs:
  claude:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          # Responds to @claude mentions in comments
```

别被 YAML 吓到，**它就四块，逐块翻译一遍你立刻懂**：

**第一块 `name`**：这张值班表的名字，随便起，会显示在 GitHub 的 Actions 页面。

**第二块 `on`（什么时候上班）**：这是触发条件。这里写的是「issue 评论被创建」（`issue_comment` 的 `created`）和「PR 代码行评论被创建」（`pull_request_review_comment` 的 `created`）——**也就是说，有人在 issue 或 PR 里发了条新评论，这张表就被激活**。

**第三块 `jobs`（干什么活）**：这里定义一个叫 `claude` 的任务，`runs-on: ubuntu-latest` 是说「在 GitHub 提供的最新版 Ubuntu 机器上跑」——你的代码全程待在 GitHub 的 runner 上，这也是官方说的「默认安全」。

**第四块 `steps`（具体步骤）**：核心就一步——`uses: anthropics/claude-code-action@v1`，意思是「用 Anthropic 官方做的这个 action，版本 v1」。下面 `with` 里把 `ANTHROPIC_API_KEY` 密钥传进去（密钥怎么来下一节专门讲）。那行 `# Responds to @claude mentions` 是注释，提醒你这份配置的作用就是「响应评论里的 @claude」。

**最妙的一点在这儿**：这份配置里，你**没看到任何地方写「只在评论包含 @claude 时才跑」**。为啥不用写？因为 v1 版本会**自动检测模式**。官方原话：

> 该 action 现在根据您的配置自动检测是在交互模式（响应 `@claude` 提及）还是自动化模式（立即使用提示运行）下运行。

说白了：**你给了 `prompt` 参数，它就「自动化模式」——一触发立刻照着 prompt 干**（下一节的 code review 就是这种）；**你没给 `prompt`，它就「交互模式」——只在评论里出现 `@claude` 时才响应**。这份最小配置没写 `prompt`，所以它自动进交互模式，乖乖等你 @。

> ⚠️ 这是从 beta 版升级过来的人最容易翻车的地方。**老版本要手动写 `mode: "tag"`，还要用 `direct_prompt`**；v1 把这些全砍了——`mode` 删掉（自动检测）、`direct_prompt` 改叫 `prompt`。如果你在网上抄到带 `@beta`、`mode:`、`direct_prompt` 的老配置，**直接换成 v1 写法**，别照抄。

那 `--max-turns`、`--model` 这些 CLI 参数（前面几篇见过的）往哪塞？官方给了个统一出口叫 `claude_args`——**你本地能用的 Claude Code CLI 参数，基本都能从这儿传进去**：

```yaml
- uses: anthropics/claude-code-action@v1
  with:
    anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
    prompt: "Your instructions here"          # 可选：给它的指令
    claude_args: "--max-turns 5 --model claude-sonnet-4-6"  # 可选：CLI 参数
```

常用的几个 `claude_args`，官方列了这些：

| 参数 | 干什么 | 默认 |
|------|--------|------|
| `--max-turns` | 最多来回几轮（防它没完没了烧钱） | 10 |
| `--model` | 用哪个模型（如 `claude-opus-4-8`） | 默认 Sonnet |
| `--allowedTools` | 允许用哪些工具（逗号分隔） | — |
| `--mcp-config` | MCP 配置文件路径（第 22 篇那套） | — |
| `--debug` | 开调试输出，排错时用 | 关 |

补一个官方明确写在文档里的事实：**Claude Code GitHub Actions 默认用 Sonnet**；想用 Opus 4.8，得在 `claude_args` 里加 `--model claude-opus-4-8` 显式指定。日常 review、改小 bug，Sonnet 完全够；真碰到要它啃大改动、做复杂推理，再换 Opus（模型怎么挑，第 30 篇那套「派活挑人」的逻辑在这儿照样成立）。

> 💡 一句话总结：workflow 就是给 Claude 排的**值班表**——`on` 定**何时上班**、`steps` 里 `claude-code-action@v1` 定**干什么**；v1 会**自动检测模式**（给 `prompt` 就自动跑、不给就等 `@claude`）；CLI 参数统一从 **`claude_args`** 塞进去。

---

## 05 三个真用得上的用例：自动审、自动改、定时跑

光会响应 `@claude` 还只是入门。**这一节给你三份现成的 workflow**，对应三种最高频的场景。每一份都标清「它解决什么、配置长啥样」。

### 用例一：每个 PR 自动 code review（不用 @，自动跑）

这是最常用的一个，也是开头那个半夜场景的主角。**它不用任何人 @，只要有 PR 开出来或更新，Claude 就自动审一遍。**

```yaml
name: Code Review
on:
  pull_request:
    types: [opened, synchronize]
jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Review this pull request for code quality, correctness, and security."
          claude_args: "--max-turns 5"
```

跟上一节那份比，差别就两处：**触发条件换成了 `pull_request` 的 `opened`（PR 刚开）和 `synchronize`（PR 有新提交）**；而且**多给了一个 `prompt`**。还记得上一节说的「给了 prompt 就自动化模式」吗？——正因为这里写了 `prompt`，它一触发就直接照着「审查代码质量、正确性和安全性」去干，**不等任何人 @**。每次有人推代码上来，它都自动过一遍，评论区里就躺好了反馈。

想要更省心、连 workflow 都不想写的「全托管」自动 review？Anthropic 还有个独立的 **Code Review** 服务（面向 Team / Enterprise 订阅），开关一开每个 PR 自动审、按严重程度标记、还能用 `REVIEW.md` 定制。本篇讲的是「在你自己的 CI 里跑 Claude」，那套托管服务是另一条路，这里点到为止。

### 用例二：按 issue 自动改（@claude 派活）

这个走的是上一节那份「响应 @claude」的最小 workflow，**重点在你评论里怎么说**。issue 描述写清楚后，在评论里：

```text
@claude 按这个 issue 的描述把功能实现了
```

它会读 issue 描述、读相关代码、读 `CLAUDE.md`，然后**在一个新分支上把功能实现，并开一个 PR 供你审查**。我给一个内部小工具加过一个导出 CSV 的功能，就是这么干的——issue 里把字段、格式写明白，@ 它一句，二十来分钟后 PR 就开好了，审完直接合，**从头到尾没打开过编辑器**。

修 bug 同理：

```text
@claude 把用户面板组件里那个 TypeError 修了
```

它定位错误、改掉、更新分支或开新 PR。

### 用例三：定时任务（到点自己跑）

这个连「事件」都不需要，**纯按时间触发**——比如每天早上九点生成一份昨日提交和待办 issue 的汇总。

```yaml
name: Daily Report
on:
  schedule:
    - cron: "0 9 * * *"
jobs:
  report:
    runs-on: ubuntu-latest
    steps:
      - uses: anthropics/claude-code-action@v1
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
          prompt: "Generate a summary of yesterday's commits and open issues"
          claude_args: "--model opus"
```

`on: schedule` 配 `cron: "0 9 * * *"` 就是「每天 9 点（UTC）」跑一次，触发后照着 `prompt` 生成汇总。这种「无人值守、定时产出」的活，**最适合丢给这个夜班同事**。

三个用例，三种触发，横着对比一眼看清：

| 用例 | 触发方式 | 给 prompt 吗 | 是什么模式 | 典型场景 |
|------|---------|-------------|-----------|---------|
| **自动 review** | `pull_request`（开/更新 PR） | 给 | 自动化（自动跑） | 每个 PR 自动审一遍 |
| **按 issue 改** | `issue_comment`（评论 @claude） | 不给 | 交互（等 @claude） | issue 转成代码、修 bug |
| **定时任务** | `schedule`（cron 到点） | 给 | 自动化（自动跑） | 每日报告、定期巡检 |

记住那条贯穿三者的主线：**给 `prompt` = 它自动照着干；不给 `prompt` = 它等你 `@claude`。** 想清楚你要哪种，就知道该不该写 prompt。

> 💡 一句话总结：三个最实用的用例——**PR 自动 review**（给 prompt，自动跑）、**按 issue 自动改**（不给 prompt，@claude 触发）、**定时任务**（cron + prompt，到点自动产出）；本质都是「触发方式 + 给不给 prompt」的组合。

---

## 06 密钥安全：那条绝不能踩的红线

到这儿你一定注意到了，前面每份 workflow 里都有这么一行：

```yaml
anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**这一节专讲这一行背后的安全规矩——它是整篇里最不能马虎的地方。** 第 04 篇我们配过本地的 `ANTHROPIC_API_KEY`，还说过「具体到 CI 的玩法第 44 篇再讲」，就是现在了。

先把官方那句加了红框的警告原封不动放这儿：

> 永远不要直接将 API 密钥提交到您的仓库。

为啥这是红线？**因为 GitHub 仓库——尤其公开仓库——是全世界都能看的。** 你要是图省事，把真实的 `sk-ant-xxxx` 直接写进 YAML 文件提交上去，等于**把你的钱包密码贴在了公告栏上**：有人扫到就能拿你的 key 疯狂调 API，账单全算你头上。这种事在 GitHub 上天天发生，扫密钥的爬虫一刻不停。

正确做法就一条：**用 GitHub Secrets，绝不硬编码。** 官方给的步骤很清楚：

> - 将您的 API 密钥添加为名为 `ANTHROPIC_API_KEY` 的仓库密钥
> - 在工作流中引用它：<code v-pre>anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}</code>

**类比：把钥匙锁进保险柜，YAML 里只留个取钥匙的暗号。** GitHub Secrets 就是仓库自带的保险柜——你把真实 key 锁进去，它加密存着、不会显示在任何日志或界面里。workflow 文件里你写的 <code v-pre>${{ secrets.ANTHROPIC_API_KEY }}</code> 不是密钥本身，**只是一句「去保险柜把那把叫 ANTHROPIC_API_KEY 的钥匙取出来用」的暗号**。文件可以大大方方提交、公开，因为里面压根没有真东西。

怎么把 key 存进这个保险柜？手动的话：打开仓库的 **Settings → Secrets and variables → Actions**，点 **New repository secret**，名字填 `ANTHROPIC_API_KEY`，值填你的真实 key（从 [Claude Console](https://console.anthropic.com) 拿，第 04 篇讲过）。如果你前面用了 `/install-github-app`，这一步它已经替你办好了。

这套和我们一路强调的安全主线是一脉相承的。第 21 篇讲过**提示注入**——有人往 issue、PR 评论里藏「写给 AI 看的恶意指令」骗 Claude 执行。**云端这套尤其要警惕这一点**：你的 Claude 现在会自动去读 issue 和 PR 评论的内容，而这些内容**任何陌生人都能提交**。所以官方的几条最佳实践，每一条都值得照做：

| ❌ 危险做法 | ✅ 安全做法 |
|-----------|-----------|
| 把真实 key 写进 YAML 提交 | 存进 GitHub Secrets，YAML 只引用 |
| 把权限开到最大图省事 | 只授必要的权限（Contents / Issues / PR 三项读写） |
| Claude 开的 PR 直接合 | **合并前自己审一遍** |
| 公开仓库随便让陌生人 @claude 派活 | 留意提示注入，敏感仓库收紧触发条件 |

最后一行的「合并前自己审一遍」值得专门强调——**Claude 干得再好，它开的 PR 也只是「一个贡献者的提交」，不是免审金牌**。官方原话：「在合并前审查 Claude 的建议」。一条值得守住的铁规矩是：**云端 Claude 开的 PR，一律当成实习生交的活来审**，绝不因为「是 AI 写的看着挺像样」就闭眼合。

> 💡 一句话总结：密钥红线就一条——**绝不把真实 key 写进仓库**，而是存进 **GitHub Secrets**（加密保险柜），YAML 里只用 <code v-pre>${{ secrets.ANTHROPIC_API_KEY }}</code> 这个暗号引用；再配上「最小权限 + 合并前必审 + 警惕提示注入」三条，才算把云端这套用得稳。

---

## 07 动手：5 分钟把它装进你自己的仓库

光看不练等于没学。下面带你**真正把它装进一个你自己的 GitHub 仓库并验证跑通**。找一个**你有管理员权限、且不重要的测试仓库**练手（别拿生产仓库试），全程不依赖你已有的复杂环境。

> 前提：你得是这个仓库的管理员、走的是直接 Claude API（不是 Bedrock / Vertex）、本地装好了 Claude Code。这几步要连 GitHub 和 Claude 服务器，国内网络不通就先开「魔法上网」。

**第一步：在本地 Claude Code 里跑安装命令**

进到那个测试仓库的目录，启动 `claude`，然后敲：

```text
/install-github-app
```

**预期**：它会拉起一个引导，通常会让你在浏览器里点开授权——给 Claude GitHub App 选仓库、确认那三项权限（Contents / Issues / Pull requests 读写），再引导你把 `ANTHROPIC_API_KEY` 配进仓库 Secrets。**跟着提示一步步点完，看到它提示安装成功、workflow 文件已写入即可。**

**第二步：确认 workflow 文件到位**

去你仓库里看一眼，应该多了个文件：

```text
.github/workflows/claude.yml
```

**预期**：这个文件存在，内容大致就是第 04 节那份最小 workflow（`on: issue_comment` + `claude-code-action@v1`）。**看到它 = 值班表已经排上了。** 如果没有，说明上一步没走完，回去重跑 `/install-github-app`。

**第三步：确认密钥进了保险柜**

去仓库 **Settings → Secrets and variables → Actions** 看一眼。

**预期**：Repository secrets 列表里有一项 `ANTHROPIC_API_KEY`（只显示名字，值是加密的看不到）。**看到这个名字 = 钥匙锁进保险柜了。**

**第四步：开个测试 issue，@ 它一句**

在仓库里新建一个 issue，标题随便，正文里写一句具体的小任务，比如：

```text
@claude 在 README 末尾加一行 "Hello from Claude Code GitHub Actions"，然后开个 PR
```

**预期**：
- 去仓库的 **Actions** 标签页，你会看到一个名为 `Claude Code` 的 workflow **正在运行**（转圈）或已完成（绿勾）。
- 等一两分钟（它要在 runner 上启动、读仓库、干活），**Claude 会在这个 issue 下回一条评论，并开出一个 PR**——PR 里就是给 README 加的那一行。
- **看到这个自动开出来的 PR = 整条链路通了。**

如果等了几分钟啥都没发生，按官方故障排除挨个查：**评论是不是打成了 `/claude`（必须是 `@claude`）、App 是不是装好了、Secrets 里 key 在不在、Actions 是不是被禁用了**。最常见的就是栽在 `/claude` 上，别重蹈覆辙。

**第五步：审完再合（别闭眼合）**

打开那个 PR，**像审实习生的活一样看一眼 diff**，确认它真就加了那一行、没动别的。确认无误再合。

跑通这五步，你就把「装 App → 配密钥 → 排值班表 → @ 它派活 → 它自动开 PR → 你审完合」这条完整链路**亲手走了一遍**。以后给任何仓库配，本质都是这套流程，无非换换 workflow 里的触发条件和 prompt。

> 💡 一句话总结：动手就五步——**`/install-github-app` 装、看 `claude.yml` 到位、看 Secrets 里有 key、开 issue @claude 派个小任务、确认它自动开 PR 后审完再合**；跑通这条链路，比记十条配置都顶用。

---

## 08 小结

这一篇我们把 Claude Code 送上了云端——**从「你在场它才干」到「一句 `@claude` 它自己跑」，全靠 GitHub Actions 这套自动化**。

把核心要点串起来回顾：

| 你要做的事 | 用什么 | 关键点 |
|-----------|--------|--------|
| 理解它是什么 | 住进 GitHub 的 Claude Code | 仓库事件触发，不用你在场，照读 `CLAUDE.md` |
| 喊它干活 | 评论里 `@claude` + 具体要求 | 认 **@ 符号**，不是 `/claude` |
| 装上它 | 本地跑 `/install-github-app` | 要管理员权限 + 直接 API；给三项读写工牌 |
| 配触发与行为 | `.github/workflows/` 的 YAML | `on` 定何时、`claude_args` 传参；v1 自动检测模式 |
| 选用例 | review / 按 issue 改 / 定时 | **给 prompt 自动跑，不给等 @claude** |
| 保护密钥 | GitHub Secrets | **绝不硬编码**，YAML 只用 <code v-pre>${{ secrets.* }}</code> 引用 |

**你现在应该能：** 说清 GitHub Actions 版 Claude Code 和本地版的区别、在 PR / issue 里用 `@claude` 派活（还不会打成 `/claude`）、看懂并改一份最小 workflow YAML、按「自动 review / 按 issue 改 / 定时」三种需求挑对配置、把 API key 安全地锁进 GitHub Secrets，并亲手把这套装进自己的仓库验证跑通。**这套云端自动化，是你让 Claude 从「桌上的助手」升级成「团队里 24 小时在岗的一员」的那一步。**

配好 Actions 之后，就再没必要为「PR 没人 review」熬夜——**夜班同事顶上了，你睡你的。**

> 顺带一提：如果你们用的是 GitLab 而不是 GitHub，Claude 也有对应的 **GitLab CI/CD** 集成（目前是测试阶段），思路一模一样——在 `.gitlab-ci.yml` 里加个作业、配个掩码变量、`@claude` 触发。这一篇讲透了 GitHub 这条，GitLab 那条照着官方文档套就行。

---

下一篇 **45「Agent SDK」**——这一篇的 GitHub Actions，官方说它「建立在 Claude Agent SDK 之上」。换句话说，`@claude` 自动开 PR 这套能力，底层是一个能让你**用代码把 Claude Code 嵌进任何程序**的 SDK 在撑着。GitHub Actions 只是它最现成的一个封装。下一篇就掀开这层盖子：**如果你想搞的自动化超出了 `@claude` 那几招——比如做个自己的 AI 客服、批量处理一万个文件、把 Claude 塞进你自家的后台系统，该怎么用代码直接驱动它？** 想想看：GitHub Actions 帮你把「触发」和「跑」都封装好了，可一旦你想自己定义「什么时候跑、跑完结果往哪送」，是不是就得往下挖一层了？
