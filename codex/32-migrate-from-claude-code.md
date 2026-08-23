---
seoTitle: "从 Claude Code 迁移到 Codex 完整指南"
description: "把 CLAUDE.md、Skill、MCP、Hook、权限和日常工作流迁移到 Codex 的对应方式，指出可以复用与必须重写的部分，并给出适合中文开发者直接照做的操作思路、检查方法与风险边界。"
published: "2026-06-12"
lastVerified: "2026-06-20"
author: stormzhang
officialSources:
  - https://developers.openai.com/codex/migrate
related:
  - ./34-capstone.md
  - ./31-speed.md
relatedPages:
  - title: "Codex 综合实战：从需求到 Git 提交"
    link: ./34-capstone.md
  - title: "Codex 提速指南：上下文、模型与并行策略"
    link: ./31-speed.md
---
# 32 · 从 Claude Code 迁移：旧地图换个工具，照样能找到家

> 📚 **系列导航**：上一篇〔[31 进阶技巧与提速](31-speed.md) 〕讲怎么把整条工作流跑得更快、更省、更少返工。这一篇换个角度——专门写给**从 Claude Code 转过来的你**：你脑子里那套用了大半年的心智模型，哪些能直接搬、哪些得改个名、哪些是 Codex 独有的新东西。下一篇〔[33 Windows 使用要点](33-windows.md) 〕再聊 Windows 上那些专属的坑。

先还原一段真实对话。上个月一个一直用 Claude Code 的朋友装好 Codex，开口就是一连串问号：

> 他：「Codex 的 `CLAUDE.md` 放哪？我项目根目录那份它怎么不读？」
> 我：「Codex 不认 `CLAUDE.md` 这个名，它读 `AGENTS.md`，内容你几乎能原样搬过去。」
> 他：「那我 `settings.json` 里那套 `allow` / `deny` 权限规则呢？」
> 我：「也得换——Codex 用 `~/.codex/config.toml`，TOML 格式，权限那套思路是『沙箱 + 审批』，不是 `allowedTools`。」
> 他：「`claude -p` 跑脚本那套呢？还有 `/compact`、`/clear` 这些命令在不在？」
> 我：「`claude -p` 对应 `codex exec`，斜杠命令大半都在、名字基本一样。说白了你 90% 的肌肉记忆能直接用。」

他听完那口气一下松了——**原来不是从头学一个新工具，是同一张地图换了套地名**。这一篇就把他问的、和他没来得及问的，一次性给你对齐：哪些概念一一对应、哪些差异照搬会栽跟头、最后带你把一份真实的 `CLAUDE.md` 手动改写成 `AGENTS.md`。

**看完这一篇，你会拿到：**

- 一颗定心丸：你 90% 的 Claude Code 心智模型能直接搬到 Codex，迁移成本比想象低
- 一张「Claude Code 概念 → Codex 对应」的完整对照表，逐项讲清关键差异
- 项目说明书、配置文件、权限模型三处「名字像但脾气不同」的地方，照搬必踩的坑
- 哪些 Claude Code 的东西在 Codex 里**没有、或不一样**，别想当然
- 一套照着做就能把 `CLAUDE.md` 迁成 `AGENTS.md` 的动手流程

> ⚠️ 下文凡涉及具体命令、配置键、默认行为，都以 Codex [官方文档](https://developers.openai.com/codex) 为准；模型名、版本号这类随版本变的东西，以你本地 `/model` 面板和 `codex --help` 实际显示为准，别背名字。两边工具都在快速迭代，对照表讲的是「概念怎么映射」，不是逐字保证。

---

## 01 先给定心丸：你的心智模型 90% 能直接搬

先把最容易焦虑的事说死：**从 Claude Code 转 Codex，你不是从零学一个新工具，是把同一套底层逻辑换个壳子继续用。**

为什么这么有底气？因为这俩在最根本的层面是**同一类东西**——都是终端里的 AI 编程 CLI，都跑「代理循环（agentic loop）」，都能直接读写你真实的代码库。这三点你在 Claude Code 篇里反复见过，到 Codex 这边一个字都不用改。

**类比：从安卓换到另一个牌子的安卓机。** 不是从安卓换苹果那种推倒重来——你拨号、发消息、装应用、滑动返回的肌肉记忆全在，只是图标挪了位置、设置菜单换了层级、个别 App 名字不一样。你需要的不是重新学「怎么用手机」，是花十分钟摸清「这台机器把东西放哪了」。Codex 之于 Claude Code，就是这种关系。

具体哪些能直接搬？我自己从 Claude Code 转过来时，下面这些**一次都没卡过**：

- **「想 → 做 → 看」的代理循环**：你描述需求、它制定计划、动手改、再回看结果，这套节奏完全一致。
- **「先让它读、看懂方案再放手」的习惯**：接手陌生项目先只读通读、出方案、确认后才动手，两边都该这么干。
- **「项目规矩写进一份文件、每轮开工自动读」的思路**：只是文件名和加载细节有别（下一节细说）。
- **斜杠命令当「会话控制台」**：切模型、清上下文、看状态，入口都是一个 `/`。

我第一次正式拿 Codex 干活，是去年把一个 FastAPI 项目从 Claude Code 切过来。我几乎是凭着旧习惯瞎按的——`/status` 看配置、`/model` 切模型、改之前先让它出方案——**结果八成操作直接命中**，剩下两成是文件名和权限设法不一样，翻了下官方文档十分钟就对齐了。那种「咦这我会」的熟悉感，比我预想的强太多。

> 💡 一句话总结：Codex 和 Claude Code 是同一类工具（终端 CLI + 代理循环 + 读写真实代码库），**你 90% 的心智模型能直接搬**，要重新认的只是「东西放哪、叫什么名」。

---

## 02 一张大对照表：旧地名 → 新地名

这一节是全篇的骨架——把你在 Claude Code 篇学过的每个核心概念，对到 Codex 这边的名字和位置。**先看表建立全局印象，后面几节再把差异最大的几行单独拆开讲。**

| Claude Code | Codex | 关系 | 一句话差异 |
|---|---|---|---|
| 项目说明书 `CLAUDE.md` | `AGENTS.md` | 换名 | 概念一致，发现链 / 覆写规则不同（见 03） |
| 配置文件 `~/.claude/settings.json`（JSON） | `~/.codex/config.toml`（TOML） | 换格式 | JSON → TOML，键名和结构全不一样（见 04） |
| 权限模式 + `allow`/`ask`/`deny` 规则 | 沙箱 `sandbox` + 审批 `approval` | 换思路 | 从「按工具列白名单」变「圈地 + 出圈才问」（见 05） |
| 无头模式 `claude -p` | 非交互命令 `codex exec` | 换名 | 都是「不进交互界面、跑完就走」 |
| `CLAUDE.md` 分层（用户 / 项目 / 子目录） | `AGENTS.md` 分层（全局 / 项目逐级） | 对应 | 思路一致，Codex 多了 `AGENTS.override.md` |
| 外部工具 MCP | MCP | 几乎通用 | 同一套协议，配置写法各自不同 |
| 子代理 Subagents | 子代理 Subagents | 对应 | 两边都有，配置位置不同 |
| 技能 Skills | 技能 Skills | 对应 | 两边都有，本地目录组织略有差异 |
| 斜杠命令（`/model`、`/compact`…） | 斜杠命令（`/model`、`/compact`…） | 大半同名 | 命令名基本一致，个别有出入（见 06） |
| 自动记忆 memory（默认开） | Memories / Chronicle | 对应但脾气不同 | Codex 记忆**默认关**、有地区限制、异步生成 |
| 模型 Opus / Sonnet / Haiku | GPT-5.x 系列 | 换型号 | 旗舰 `gpt-5.5`、轻量 `gpt-5.4-mini` 等 |

这张表你不用背，**记住一个总规律就行**：

**凡是「概念」层面的东西（项目说明书、权限、记忆、子代理、技能、MCP），两边都有，迁移就是「换名 + 调写法」；真正会让你栽跟头的，是那几个「名字像、脾气不同」的——项目说明书、配置文件、权限模型。** 下面三节专治这三个。

至于模型对照，一句话带过：你在 Claude Code 那边按「Opus 啃硬活、Sonnet 跑日常、Haiku 跑杂活」选型号，到 Codex 这边换成「`gpt-5.5` 啃硬活、`gpt-5.4-mini` 跑杂活」，选型逻辑（拿任务难度匹配算力）一模一样，详见〔[30 怎么选模型](30-models.md) 〕。注意别把 `gpt-5.4` 当旗舰——旗舰是 `gpt-5.5`，`gpt-5.4-mini` 是它的轻量款。

> 💡 一句话总结：概念层面两边几乎一一对应，迁移多是「换名 + 调写法」；**真正的坑集中在项目说明书、配置文件、权限模型这三处「名字像但脾气不同」的地方**，下面逐个拆。

---

## 03 项目说明书：`CLAUDE.md` → `AGENTS.md`

这是你迁移时第一个会撞的问题，也是我朋友开口第一问：**Codex 不读 `CLAUDE.md`。** 你项目根目录那份它视而不见，因为它认的文件名叫 `AGENTS.md`。

好消息是：**内容你几乎能原样搬。** 项目概述、技术栈、常用命令、代码约定、禁改清单——这五类东西在两边都是同样的写法、同样的目的。你在 Claude Code 篇 [18] 学的「写每轮该记住的事实、删一切看代码就能自证的东西」，到 `AGENTS.md` 这边一字不改照样成立。

**类比：换公司后重写一份交接文档。** 你跳槽了，旧东家那份《项目说明》里的核心信息——技术栈、怎么跑测试、哪些雷区别碰——大半能直接复制到新东家的模板里。但新公司文档归档的**文件夹结构和命名规范**不一样，你得按它的来。`AGENTS.md` 就是这种「内容能搬、归档规则得换」的情况。

具体到「归档规则」的差异，有几处照搬会出问题，列清楚：

| 维度 | Claude Code（`CLAUDE.md`） | Codex（`AGENTS.md`） |
|---|---|---|
| 用户级放哪 | `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` |
| 项目级放哪 | `./CLAUDE.md` 或 `./.claude/CLAUDE.md` | `./AGENTS.md`（项目根） |
| 子目录级 | 任意子目录 `CLAUDE.md`，读到才加载 | 从 Git 根逐级到当前目录，每目录挑一个 |
| 临时覆写 | 本地变体 `CLAUDE.local.md`（不进 git） | **`AGENTS.override.md`**（那一层跳过同级 `AGENTS.md`） |
| 大小红线 | 按**行数**（建议 200 行内） | 按**字节**（合并后默认 32 KiB，`project_doc_max_bytes`） |

三个最该记住的差异：

**第一，临时覆写机制不一样。** Claude Code 用 `CLAUDE.local.md` 放「只你自己看、不进 git」的本地偏好。Codex 那套是 `AGENTS.override.md` ——它存在的那一层，同级的 `AGENTS.md` 被整个跳过。这俩**不是一回事**：前者是「附加个人内容」，后者是「这一层用我、别用旁边那份」。详见 [11] 。

**第二，大小红线从「数行」变「数字节」。** Claude Code 劝你 `CLAUDE.md` 别超 200 行；Codex 是硬指标——合并后超过默认 32 KiB 就开始截断、甚至整份文件被拦在外面。我迁移那份 FastAPI 项目的说明书时就吃过一记：原来 `CLAUDE.md` 写得偏肥，搬过来一开始没在意，后来发现子目录那份指令没生效，一查才知道是合并体积撞了上限。所以**搬之前顺手瘦个身，删掉那些 Codex 看代码就能自证的内容**。

**第三，发现链是「拼接」不是「覆盖」。** 全局那份和项目那份**同时生效**，冲突时离当前目录越近的越优先。这点和 Claude Code 的层级加载思路一致，但 Codex 的合并顺序（从根到叶拼接、越近越晚拼、优先级越高）官方写得更明确，照 [11] 那套理解就行。

> 💡 一句话总结：`CLAUDE.md` 的**内容**几乎能原样搬进 `AGENTS.md`，但**文件名、临时覆写机制（`CLAUDE.local.md` → `AGENTS.override.md`）、大小红线（行数 → 字节）** 三处不同，搬之前顺手瘦身。

---

## 04 配置文件：`settings.json`（JSON）→ `config.toml`（TOML）

第二个坑：你 `~/.claude/settings.json` 里那套配置，**不能直接复制到 Codex**。两边连文件格式都不一样。

- Claude Code：`~/.claude/settings.json` ，**JSON 格式**，权限、环境变量、Hook、默认模型都塞这一个文件。
- Codex：`~/.codex/config.toml` ，**TOML 格式**，模型、沙箱、审批、MCP 等行为旋钮都在这儿。

**类比：同样是装修配电图，一个用公制一个用英制。** 图纸要表达的东西（哪条线管哪个开关）是一回事，但标注的单位和符号体系是另一套。你不能拿英制图纸直接套到公制施工队手里——得逐项换算重画。JSON 配 → TOML 配，就是这种「意图一致、写法重画」的活儿。

格式差异最直观的几处，对照感受一下：

| 你想配的事 | Claude Code（`settings.json`，JSON） | Codex（`config.toml`，TOML） |
|---|---|---|
| 默认模型 | `"model": "claude-..."` | `model = "gpt-5.5"` |
| 权限 / 安全 | `"permissions": { "deny": [...] }` | `sandbox_mode = "..."` + `approval_policy = "..."` |
| 嵌套结构 | 大括号 `{ }` + 逗号 | 节表头 `[section]` + 等号 |
| 字符串引号 | 双引号必带 | 双引号 |

举个最小的对照。Claude Code 里设默认模型（JSON）：

```json
{
  "model": "claude-sonnet-4"
}
```

Codex 里设默认模型 + 推理强度（TOML）：

```toml
# ~/.codex/config.toml
model = "gpt-5.5"
model_reasoning_effort = "medium"
```

注意 TOML 的几个习惯，从 JSON 转过来最容易写错：**键值对用 `=` 不用 `:` ；不用大括号包整个对象，靠 `[section]` 这种节表头分组；行尾不加逗号。** 我刚转过来时手贱在 `config.toml` 里给每行末尾加了逗号（JSON 肌肉记忆），Codex 直接报配置解析错——这是 JSON 转 TOML 最高频的低级失误，提前告诉你，省得你也踩。

`config.toml` 里那一大堆旋钮（模型、沙箱、审批、推理强度、服务层级、MCP……）的完整讲解在〔[18 config.toml 配置详解](18-config.md) 〕，这里你只需建立一个认知：**别想着「翻译」整个 `settings.json` ，而是照着 [18] 那张键表，把你真正在用的那几条重新配一遍。** 大多数人实际改的也就模型、沙箱、审批这三五行。

> 💡 一句话总结：`settings.json`（JSON）→ `config.toml`（TOML）是**换格式重写**，不是复制粘贴；TOML 用 `=`、用 `[section]`、行尾不加逗号——别把 JSON 的逗号习惯带过来；照 [18] 那张键表重配你真在用的那几条即可。

---

## 05 权限模型：权限模式 → 沙箱 + 审批

这是迁移时**心智差异最大**的一处，也是最值得花时间重建认知的地方。照搬 Claude Code 那套「列白名单」的思路过来，你会找不着北。

Claude Code 的权限模型，核心是**两件事**：

- **权限模式（permission mode）**：六档，从 `default`（步步问）到 `bypassPermissions`（全放开），用 `Shift+Tab` 在模式间切。
- **规则白名单**：在 `settings.json` 里写 `allow` / `ask` / `deny`，按工具、按命令精确控权（比如 `deny` 掉 `rm -rf`）。

Codex 不是这套。它把「能动多大」和「问不问你」**拆成两个独立旋钮**：

- **沙箱（sandbox）**：管「能动多大」——`read-only` / `workspace-write` / `danger-full-access` 三档。
- **审批（approval）**：管「问不问你」——`untrusted` / `on-request` / `never` 三档。

**类比：从「逐项审批的报销制度」换成「划定预算 + 超支才报批」。** Claude Code 像老式报销——每一笔（每个工具、每条命令）你都预先规定准不准、要不要问，列得很细。Codex 像现代预算制——先给你圈一块「预算范围」（沙箱），范围内你随便花不打扰（审批不触发），**只有要超出这块范围时才停下来报批**。前者按「项」管，后者按「圈」管，这是思路上的根本切换。

两边映射关系，我整理成一张迁移对照表：

| 你在 Claude Code 想要的效果 | Claude Code 怎么设 | Codex 怎么设 |
|---|---|---|
| 只让它读、别动我东西 | `default` 模式 / 只读 | 沙箱 `read-only` |
| 项目里放手改、出项目要问 | `acceptEdits` 一类 | 沙箱 `workspace-write` + 审批 `on-request`（日常黄金组合） |
| 全自动、啥都别问 | `bypassPermissions` | 沙箱 `danger-full-access` + 审批 `never`（`--yolo` 更彻底：完全绕开沙箱） |
| 拦死某条危险命令 | `permissions.deny` 写规则 | rules（实验性）用 `prefix_rule()` 匹配命令前缀，`decision = "forbidden"` 即拦死 |
| 切换松紧 | `Shift+Tab` | 会话里 `/permissions` 或启动加 `-s` / `-a` |

几个搬过来必须重建的认知：

**第一，「不问」≠「放权」。** Claude Code 那套里模式越往后越松；Codex 这俩旋钮是独立的——你完全可以「只读 + 不问我」（`read-only` + `never`），意思是「随便读、但读的时候一句都别打扰」。`never` 是「不弹审批」，不是「放开权限」。

**第二，默认档跟「有没有 Git」挂钩。** Codex 启动时会智能选档：有 Git 管的目录给你 `workspace-write` + `on-request`（Auto 档），没 Git 的目录默认 `read-only` 。这是 Claude Code 没有的贴心设计，也是道安全网——别在没 Git 的临时目录里手贱开完全访问（我在 [15] 讲过自己怎么因为这个后背发凉）。

**第三，`workspace-write` 下网络默认是关的、`.git` 是只读保护的。** 这俩默认值反直觉，照搬 Claude Code 经验最容易踩。想联网得自己开 `network_access` 。

权限这块的完整玩法（三档怎么组合、rules 怎么写、`--yolo` 红线在哪）在〔[15 权限、沙箱与审批](15-permissions.md) 〕，迁移时重点是先把「两个独立旋钮」这个新心智建立起来，别拿「列白名单」的旧框架硬套。

> 💡 一句话总结：Claude Code 的「权限模式 + `allow`/`deny` 白名单」到 Codex 变成「沙箱（能动多大）+ 审批（问不问你）两个独立旋钮」；记住「不问 ≠ 放权」、默认档看有没有 Git、`workspace-write` 网络默认关——这三点照搬必栽。

---

## 06 交互习惯：斜杠命令、会话管理的异同

好消息回来了：**进了会话之后，你的肌肉记忆大半能直接用。** 斜杠命令两边重叠度极高，常用的那批几乎同名。

| 你想干的事 | Claude Code | Codex | 是否同名 |
|---|---|---|---|
| 切模型 | `/model` | `/model` | ✅ |
| 压缩上下文 | `/compact` | `/compact` | ✅ |
| 清屏开新对话 | `/clear` | `/clear` | ✅ |
| 看状态 / 配置 | `/status`（或 `/config`） | `/status` | 基本对应 |
| 生成项目说明书 | `/init` | `/init` | ✅（一个生成 `CLAUDE.md`、一个生成 `AGENTS.md`） |
| 看改动 diff | `/diff` | `/diff` | ✅ |
| 让它审一遍改动 | `/review` | `/review` | ✅ |
| 调权限松紧 | `Shift+Tab` 切模式 | `/permissions` | ❌ 入口不同 |

差异主要在两处：

**第一，调权限的入口不同。** Claude Code 靠 `Shift+Tab` 这个快捷键在权限模式间循环切；Codex 是敲斜杠命令 `/permissions` 弹选择器挑档。功能等价，但手感不一样——从快捷键变成了菜单选择。

**第二，`/clear` 和 `/new` 在 Codex 这边分得更细。** Codex 里 `/clear` 是「清屏 + 开新对话」，`/new` 是「开新对话但不清屏」。Claude Code 那边换任务清空主要就 `/clear` 一个（旧对话还能 `/resume` 找回）。细微差别，知道有就行。

我自己迁移过来后第一周的真实体感：**敲命令基本不用查**——`/model` 切模型、`/status` 看状态、`/compact` 压上下文全是顺手就来。唯一卡过一下的是想调权限，习惯性按 `Shift+Tab` 没反应，才想起来 Codex 得敲 `/permissions` 。完整的斜杠命令速查在〔[12 斜杠命令与快捷键](12-slash-commands.md) 〕，但凭旧习惯先用着、卡了再查，基本够用。

> 💡 一句话总结：斜杠命令两边重叠度极高（`/model`、`/compact`、`/clear`、`/diff`、`/review` 全同名），凭旧习惯先用、卡了再查；最大差异是调权限——Claude Code 按 `Shift+Tab`，Codex 敲 `/permissions` 。

---

## 07 别想当然：这些在 Codex 里没有或不一样

迁移最危险的不是「不会用」，是「**以为和 Claude Code 一样、结果不一样**」。这一节专门点出几个容易想当然的地方。

**第一，记忆系统默认状态相反。** Claude Code 的自动记忆默认**开着**，按 git 仓库分目录、开局自动读回。Codex 的 Memories **默认关着**、有地区限制、走后台异步生成，存放路径和控制方式都另一套。我那个朋友刚转过来时，以为「我跟它说一句它就记住了」——结果转头又用错包管理器，就是把 Claude Code 那套「默认有记忆」的预期带过来了。**必须每次生效的死规矩，两边都该写进项目说明书（`AGENTS.md`），别赌记忆**，详见 [19] 。

**第二，Codex 有 Claude Code 没有的东西。** 别假设功能只是「换名」，有些是新增的：

- **`AGENTS.override.md`**：那一层临时盖章用的覆写文件，Claude Code 无对应物（[11] ）。
- **Chronicle**：用屏幕内容喂记忆的能力，官方目前标为「研究预览版（research preview）」、需手动开启且有地区限制，Codex 独有（[19] ）。
- **基于 Git 的默认档选择**：启动时按「有没有 Git」自动给你选沙箱档（[15] ）。

**第三，反过来，Claude Code 的个别东西在 Codex 里形态不同。** 比如 Claude Code 那套 `CLAUDE.local.md` 本地变体，Codex 用 `AGENTS.override.md` 顶替但语义不同；权限白名单那套精确控命令，Codex 对应的是实验性的 rules（写法是 Starlark，不是 JSON 数组）。

下面这张「别想当然」清单，迁移前过一遍：

| 你可能的想当然 | 实际情况 |
|---|---|
| ❌ Codex 也读 `CLAUDE.md` | ✅ 它只读 `AGENTS.md`（除非你配 `project_doc_fallback_filenames`） |
| ❌ 记忆默认开着、说一句就记 | ✅ Memories 默认关、异步生成、有地区限制 |
| ❌ `settings.json` 复制过去就行 | ✅ 得重写成 TOML 的 `config.toml` |
| ❌ 权限按工具列白名单 | ✅ 是沙箱 + 审批两个旋钮 |
| ❌ `Shift+Tab` 切权限 | ✅ Codex 敲 `/permissions` |

> 💡 一句话总结：迁移最大风险是「想当然以为一样」——记忆**默认关**（不是开）、Codex 有 `AGENTS.override.md` / Chronicle / 基于 Git 的默认档这些独有物、权限是旋钮不是白名单；过一遍上面那张清单再动手。

---

## 08 动手：把一份 `CLAUDE.md` 改写成 `AGENTS.md`

光看不练记不住。下面用一份真实风格的 `CLAUDE.md` ，手把手迁成 `AGENTS.md` ，并验证 Codex 真读到了。跟着敲，五分钟搞定。

平台差异先说清：下面的 `mkdir` / `git init` 在 **Mac / Linux** 直接用；**Windows** 建议在 Git Bash 或 WSL 里敲，或用资源管理器手动建。路径里 `~` 指用户主目录，Windows 对应 `C:\Users\你的用户名\` 。

**第一步：准备一份待迁移的 `CLAUDE.md` 。**

建个玩具项目，放一份典型的 `CLAUDE.md`（就当是你从 Claude Code 项目里搬来的）：

```bash
mkdir migrate-demo && cd migrate-demo
git init
```

在项目根新建 `CLAUDE.md` ，贴入这份（故意带一行该删的废话，演示瘦身）：

```md
# migrate-demo 项目说明

这是一个基于 FastAPI 的订单管理后端。本项目由订单团队在 2023 年立项，
最初用 Flask，后来为了异步性能迁到 FastAPI，技术选型经过三轮评审……（一大段背景）

## 技术栈

- Python 3.11 / PostgreSQL / pytest

## 常用命令

- `pytest` —— 运行测试
- `ruff check .` —— 跑 lint

## 编程约定

- 所有函数必须有类型注解
- 字符串统一用双引号

## 禁区

- 不要改动 migrations/ 里已有的迁移文件
- 新增生产依赖前先问我
```

**预期**：项目根出现 `CLAUDE.md` 。注意那段立项背景——**Codex 写代码用不上，正是该在迁移时删掉的**。

**第二步：改写成 `AGENTS.md` 。**

新建 `AGENTS.md` ，把内容搬过来、**顺手砍掉那段立项背景**（其余几乎照搬）：

```md
# migrate-demo — 基于 FastAPI 的订单管理后端

## 技术栈

- Python 3.11 / PostgreSQL / pytest

## 常用命令

- `pytest` —— 运行测试
- `ruff check .` —— 跑 lint

## 编程约定

- 所有函数必须有类型注解
- 字符串统一用双引号

## 禁区

- 不要改动 migrations/ 里已有的迁移文件
- 新增生产依赖前先问我
```

**预期**：项目根出现 `AGENTS.md` ，比原 `CLAUDE.md` 短了一截——**砍掉的全是 Codex 看代码能自证或根本用不上的背景**。这就是「内容能搬、顺手瘦身」的活样本。

**第三步：让 Codex 复述它读到的指令，验证迁移成功。**

在项目目录里跑（这里只读总结，加 `--ask-for-approval never` 是为免去审批打扰、让输出干净——这正是官方文档用来验证 `AGENTS.md` 是否被读到的标准做法，放心用）：

```bash
codex --ask-for-approval never "Summarize the current instructions."
```

**预期**：Codex 会**回显你刚写的那几条**——技术栈、`pytest` / `ruff` 命令、类型注解、双引号、别动 migrations、别加依赖。只要它复述出来，就证明这份 `AGENTS.md` 这一轮确实被装进了上下文，**迁移成功**。

**第四步（可选）：把旧的 `CLAUDE.md` 处理掉。**

迁完确认无误，旧的 `CLAUDE.md` 留着也不影响（Codex 不读它），但容易让队友混淆。**要不要删由你决定**——这属于会改动文件的操作，自己手动处理，别让工具替你删。如果想让 Codex 顺带也认 `CLAUDE.md` 这个名（过渡期两边并存），可以在 `~/.codex/config.toml` 里加：

```toml
# ~/.codex/config.toml
project_doc_fallback_filenames = ["CLAUDE.md"]
```

加了这条，Codex 在每个目录的挑选顺序变成 `AGENTS.override.md` → `AGENTS.md` → `CLAUDE.md` ，取第一个存在且非空的——**过渡期想让老 `CLAUDE.md` 也被读到，这是最省事的办法**（改完重启 Codex 生效）。

> 💡 一句话总结：迁移 = 把 `CLAUDE.md` 内容搬进 `AGENTS.md` + 顺手砍背景，用「Summarize the current instructions」验证读到没；过渡期想让老 `CLAUDE.md` 也被认，配一行 `project_doc_fallback_filenames` 即可。

---

## 小结

这一篇把「从 Claude Code 迁到 Codex」这件事从心智到落地捋了一遍：

| 维度 | 关键结论 |
|---|---|
| 总判断 | 90% 心智模型直接搬，要重学的只是「东西放哪、叫什么名」 |
| 项目说明书 | `CLAUDE.md` → `AGENTS.md`，内容能搬，**临时覆写机制、大小红线（行 → 字节）** 不同 |
| 配置文件 | `settings.json`（JSON）→ `config.toml`（TOML），**换格式重写**，别带 JSON 的逗号习惯 |
| 权限模型 | 权限模式 + 白名单 → 沙箱 + 审批**两个独立旋钮**，「不问 ≠ 放权」 |
| 交互习惯 | 斜杠命令大半同名，最大差异是调权限（`Shift+Tab` → `/permissions`） |
| 别想当然 | 记忆**默认关**、Codex 有 `AGENTS.override.md` / Chronicle 等独有物 |

**你现在应该能**：把一份 `CLAUDE.md` 干净地迁成 `AGENTS.md` ；知道 `settings.json` 该重写成哪几行 `config.toml` ；用「沙箱 + 审批」的新框架替换掉「权限模式 + 白名单」的旧框架；并且躲开「以为和 Claude Code 一样、其实不一样」的那几个坑。**一句话——你那套用熟的 Claude Code 经验没白学，换个工具，照样能找到家。**

---

下一篇〔[33 Windows 使用要点](33-windows.md) 〕——前面好几处动手我都让 Windows 用户「在 Git Bash 或 WSL 里敲」，那到底为什么？Windows 上跑 Codex，沙箱、路径、终端这些地方有哪些专属的坑？留个小思考：本篇讲的沙箱机制，在一个原生没有 Linux 那套隔离能力的系统上，Codex 又是靠什么把缰绳攥住的？下一篇就把 Windows 这条线单独讲透。
