---
seoTitle: "Claude Code 插件参考：目录、清单与发布"
description: "插件目录结构、manifest、组件路径、版本与 marketplace 分发方式，适合需要制作、检查或发布 Claude Code 插件的读者"
published: "2026-06-12"
lastVerified: "2026-06-20"
author: stormzhang
officialSources:
  - https://code.claude.com/docs/zh-CN/plugins-reference
related:
  - ./46-dev-config.md
  - ./41-parallel-tasks.md
---
# 38 · 插件参考手册：把自己那套配置，打成一个能发出去的包

> 📚 **系列导航**：上一篇 [37 检查点（Checkpoints）](37-checkpoints.md) 教你怎么给会话「存档读档」，改坏了一键回到上一个安全点。这一篇换个层面——**第 24 篇教你「用别人的插件」，这一篇教你「造自己的插件」**：插件的目录到底怎么摆、`plugin.json` 每个字段管什么、能打包哪些组件、依赖怎么写、怎么建市场把它发给团队。这是插件的「参考手册」深度版。

给一个插件跑一下 `claude plugin details`，输出里常有一行数字值得留意：**这个插件每个会话常驻占 ~180 token，里头两个 skill 触发时各自再吃掉 ~2400 和 ~1800 token**。

就这么个看着人畜无害的小插件，光「挂在那儿什么都没干」就先占了一截工作台。这时候就能意识到：**插件不是个黑盒——它由几类明确的组件拼成，每一类占多少、放哪儿、怎么触发，全是有据可查的**。你把这套结构吃透，才谈得上「自己造一个、还造得干净」。

第 24 篇我们已经把「用别人的插件」走通了：加市场、装插件、`/reload-plugins` 生效、装前看信任。**这一篇不重复那些**，专啃更硬的一块——**插件的内部构造和开发发布**。说白了，第 24 篇是「会开车」，这一篇是「会拆发动机、还能自己攒一台」。

这一篇偏「参考手册」，信息密度会比前面高。**你不用一次记全**，先跟着把一个能跑的插件造出来、发出去，剩下的字段表当字典随用随查。

**看完这一篇，你会拿到：**

- 插件的标准目录长相，以及那条「`.claude-plugin/` 里只放 `plugin.json`」铁律背后的完整结构
- `plugin.json` 清单的字段全景：必填的、元数据、组件路径、用户配置、依赖，一张表查到底
- 插件能打包的全部组件（skill / command / agent / hook / MCP / LSP / monitor）各自放哪、有什么限制
- `${CLAUDE_PLUGIN_ROOT}` 这几个路径变量为什么必须用、不用会怎样
- 从零造一个插件、本地测、建市场、发给团队的完整链路，全程给命令和预期输出
- 版本管理和依赖这两个发布时绕不开的坑，怎么填

---

## 01 先把插件的「骨架」摆正：标准目录长相

第 24 篇你已经见过插件最简的样子——一个 `plugin.json` 加几个组件文件夹。但要自己造一个像样的插件，得先把**完整骨架**看清楚，不然加到一半就乱。

先给结论：**一个插件就是一个文件夹，里头一个「清单文件」定身份，其余按组件类型分门别类摆在根目录下。**

**类比：一套乐高积木。** 一盒乐高里有两样东西——**一本拼装说明书**，告诉你这套叫什么、有哪些零件；**几个分格的零件盒**，按类型把积木块装好（轮子一格、窗户一格）。插件就是这么个结构：`plugin.json` 是那本说明书，`skills/`、`agents/`、`hooks/` 这些文件夹是分格零件盒。**说明书有它专属的位置，零件盒全摊在外面**——这就是下面那条铁律的由来。

官方给的完整插件目录长这样（我删掉了不常用的，留核心）：

```text
my-plugin/
├── .claude-plugin/           # 元数据目录
│   └── plugin.json           # 清单（说明书）——只有它放这儿
├── skills/                   # Skills，每个一个 <名字>/SKILL.md
│   └── code-reviewer/
│       └── SKILL.md
├── commands/                 # Skills 的扁平 .md 写法（老形式）
│   └── status.md
├── agents/                   # Subagent 定义
│   └── security-reviewer.md
├── hooks/                    # Hook 配置
│   └── hooks.json
├── .mcp.json                 # MCP server 定义
├── .lsp.json                 # LSP server 配置
├── bin/                      # 加进 PATH 的可执行文件
├── scripts/                  # Hook 和工具脚本
└── settings.json             # 插件的默认设置
```

这里有条**官方用警告框圈起来、新手必踩的铁律**，我替你钉死：

> `.claude-plugin/` 目录包含 `plugin.json` 文件。所有其他目录（commands/、agents/、skills/、output-styles/、themes/、monitors/、hooks/）必须在 plugin 根目录，而不是在 `.claude-plugin/` 内。

说白了：**`.claude-plugin/` 这个文件夹里只准躺着 `plugin.json` 一个东西**，`skills/`、`agents/`、`hooks/` 全部摆在它的**外面、跟它平级**。这个坑第 24 篇就提过——第一次打包很容易手贱把 `skills/` 塞进 `.claude-plugin/`，结果插件能加载、skill 死活不出现，排查半天才发现是位置摆错。**记住乐高那个画面：说明书归说明书的盒，零件盒全摆外头。**

还有个容易被忽略的点，官方写得很明确：**插件根目录里的 `CLAUDE.md` 不会被当成项目上下文加载**。插件想给 Claude 喂指令，得通过 skill、agent、hook 这些组件，不能靠在插件里塞一个 `CLAUDE.md`。这跟第 18 篇讲的项目级 `CLAUDE.md` 是两码事，别搞混。

> 💡 一句话总结：插件 = 一本说明书（`.claude-plugin/plugin.json`）+ 根目录下分格的零件盒（各组件文件夹）；**铁律就一条——`.claude-plugin/` 里只放 `plugin.json`，别的文件夹全摆根目录**。

---

## 02 plugin.json：这份「清单」每个字段管什么

骨架摆正了，来拆那本说明书——`plugin.json`。它声明插件的身份和配置，是整个插件的核心。

**先记一个反直觉但很省心的事实：清单是可选的。** 官方原话——如果你省略 `plugin.json`，Claude Code 会自动去默认位置（`skills/`、`agents/` 这些）找组件，连插件名都从文件夹名推出来。**只有当你需要写元数据、或自定义组件路径时，才需要这份清单。** 但凡你想正经发布，肯定要写，所以咱们按写的来讲。

**类比：报关单。** 一批货物出关，得附一份报关单——里头写清这批货叫什么名字、谁发的、版本批次、装了哪些东西。海关（Claude Code）照着这份单子核对、登记、上架。`plugin.json` 就是插件的报关单：**名字、作者、版本、装了哪些组件，全在这一张纸上说清。**

### 唯一的必填字段

如果你写了清单，**只有一个字段是必填的**：

| 字段 | 类型 | 说明 |
|------|------|------|
| `name` | string | 唯一标识符，kebab-case（小写加连字符）、不带空格 |

`name` 为什么这么关键？因为**它就是组件的命名空间（namespace）前缀**。一个叫 `plugin-dev` 的插件里有个 agent 叫 `agent-creator`，在界面里会显示成 `plugin-dev:agent-creator`；skill 调用就是 `/plugin-dev:xxx`。**命名空间是插件防撞名的根本机制**——你装十个插件，各家的 skill 都带自家前缀，不会打架。

### 元数据字段（描述插件「是什么」）

这些不影响功能，但发布时该填，让用户看得明白：

| 字段 | 干什么 |
|------|--------|
| `displayName` | 界面里显示的人类可读名，能带空格大小写；省了就回退用 `name` |
| `version` | 语义版本号（semantic versioning）。**设了它，用户只在你提版本号时才收到更新**（这是个大坑，第 08 节细说） |
| `description` | 一句话说清插件干嘛，浏览/安装时显示 |
| `author` | 作者信息（`name` / `email` / `url`） |
| `homepage` / `repository` / `license` | 文档地址 / 源码地址 / 许可证 |
| `keywords` | 发现标签，帮人搜到你 |

### 组件路径字段（指明组件「在哪」）

默认情况下组件就放在 `skills/`、`agents/` 这些标准位置，**你完全可以不写这些字段**。只有当你想把组件放到非标准路径时才用：

| 字段 | 指向什么 |
|------|---------|
| `skills` | 额外的 skill 目录（**追加**到默认 `skills/`） |
| `commands` / `agents` / `outputStyles` | 自定义路径（**替换**默认目录） |
| `hooks` / `mcpServers` / `lspServers` | 配置文件路径，或直接内联写在这儿 |
| `dependencies` | 这个插件依赖的其他插件（第 08 节讲） |

这里藏着一个**特别容易栽的细节**，官方专门列了「路径行为规则」：**有的字段是「替换默认」，有的是「追加到默认」**。

- **替换默认**：`commands`、`agents`、`outputStyles`。一旦你写了 `commands`，默认的 `commands/` 目录就**不再被扫描**了。想保留默认还加别的，得显式列上：`"commands": ["./commands/", "./extras/"]`。
- **追加到默认**：`skills`。默认 `skills/` **永远会被扫描**，你在 `skills` 字段里列的目录是跟它一起加载的。

这个坑很容易踩：给一个插件加了 `"agents": ["./extra-agents/reviewer.md"]`，想着「再加一个 agent」，结果原来 `agents/` 目录里那俩 agent 全不见了——因为 `agents` 是**替换**不是追加。改成把三个都列进数组才好。**记不住就查这张规则，别凭感觉。**

一份带元数据的完整清单大概长这样：

```json
{
  "name": "deployment-tools",
  "displayName": "Deployment Tools",
  "version": "1.2.0",
  "description": "Deployment automation tools",
  "author": { "name": "Dev Team", "email": "dev@company.com" },
  "license": "MIT",
  "keywords": ["deployment", "ci-cd"]
}
```

> 💡 一句话总结：`plugin.json` 是插件的报关单，**唯一必填的是 `name`（它定命名空间）**；组件路径字段里要分清「替换默认」（commands/agents）和「追加默认」（skills），搞反了组件会凭空消失。

---

## 03 插件能打包哪些组件：七类零件认全

这是本篇最该记牢的一节——**插件这个盒子，到底能装哪七类零件**。第 24 篇粗讲过几类，这里把官方支持的全列清，标明各自放哪、有什么独有的限制。

**类比：乐高那几格零件盒，每格只装一种件。** 轮子格、窗户格、人仔格——分门别类才拼得快。插件的组件也是一格一类：

| 组件 | 放哪 | 一句话作用 | 触发方式 |
|------|------|-----------|---------|
| **Skills** | `skills/<名字>/SKILL.md` | 可调用的专项能力（第 26 篇） | 你 `/插件名:skill名`，或 Claude 自动调 |
| **Commands** | `commands/*.md` | skill 的扁平老写法，新插件用 skills | 同上 |
| **Agents** | `agents/*.md` | 专项 subagent（第 23 篇） | 出现在 `/agents`，Claude 派或你点 |
| **Hooks** | `hooks/hooks.json` | 事件触发的自动动作（第 33 篇） | 生命周期事件自动触发 |
| **MCP servers** | `.mcp.json` | 连外部服务（第 22 篇） | 启用即自动起，工具混进工具箱 |
| **LSP servers** | `.lsp.json` | 实时代码智能（跳定义、查引用） | 处理代码时自动用，需另装语言服务器 |
| **Monitors** | `monitors/monitors.json` | 后台监视日志/状态，有动静通知 Claude | 插件激活时自动起（实验性） |

几类需要单独点几句，都是官方文档里写明、但容易漏的限制：

**Agents 在插件里被「削权」了。** 这点很关键。官方明确：**出于安全原因，插件提供的 agent 不支持 `hooks`、`mcpServers` 和 `permissionMode` 这三个 frontmatter 字段。** 也就是说，插件里的 subagent 不能自己偷偷挂 hook、起 MCP、或改权限模式——这是防止你装个插件，它的 agent 在背后给你换权限。它支持的字段包括 `name`、`description`、`model`、`effort`、`maxTurns`、`tools`、`disallowedTools`、`skills`、`memory`、`background`、`isolation` 这些（`isolation` 唯一合法值是 `"worktree"`）。

**Hooks 能监听的事件巨多。** 插件 hook 跟你自己写的 hook（第 33 篇）监听的是同一批生命周期事件——从 `SessionStart`（会话开始）、`PreToolUse`（工具调用前，能拦）、`PostToolUse`（调用成功后），到 `Stop`（回答结束）、`SessionEnd`（会话终止），官方列了三十个。新手不用记全，**记住「几乎任何时机都能挂个动作」就行**，具体留到第 33 篇。

**Hook 类型不止「跑脚本」。** 除了最常见的 `command`（跑 shell 命令），还有 `http`（把事件发到一个 URL）、`mcp_tool`（调 MCP 工具）、`prompt`（用模型评估一段提示）、`agent`（跑个 agentic 验证器）。

**Monitors 是实验性的。** 它让插件在后台盯着日志或状态，有新行就当通知喂给 Claude，不用你开口让它监视。**实验性，架构可能变**，且只在交互式会话里跑、需要 Claude Code v2.1.105 以上。新手了解有这么个东西即可。

还有两类「半组件」值得知道：**`bin/` 目录**里的可执行文件，插件启用时会被加进 Bash 工具的 `PATH`，能当裸命令直接调；**`settings.json`** 是插件的默认设置，但目前只支持 `agent` 和 `subagentStatusLine` 两个键——其中设 `agent` 能让插件启用时直接把某个自定义 agent 顶成主线程，等于「装上这个插件就换了套人设」。

> 💡 一句话总结：插件能装七类零件——skill / command / agent / hook / MCP / LSP / monitor，各有固定位置；**重点记两条限制：插件里的 agent 不许带 hook/MCP/权限模式（安全削权）、monitor 是实验性的**。

---

## 04 ${CLAUDE_PLUGIN_ROOT}：为什么路径必须用变量，不能写死

这一节单拎出来，因为它是**自己造插件时最高频的报错来源**，而道理一句话能说透。

先看场景：你的插件里有个 hook 要跑 `scripts/format.sh`，或者 MCP server 要 `node server.js`。你想当然地写绝对路径 `/Users/你/my-plugin/scripts/format.sh`——**完蛋，这条路在别人机器上根本不存在**，而且就算在你机器上，**插件每次更新缓存目录都会变**。

官方的解法是给你三个**路径变量**，在 hook 命令、MCP/LSP 配置、skill/agent 内容里都会被自动替换：

| 变量 | 指向 | 用来 |
|------|------|------|
| `${CLAUDE_PLUGIN_ROOT}` | 插件安装目录的绝对路径 | 引用插件**自带**的脚本、二进制、配置 |
| `${CLAUDE_PLUGIN_DATA}` | 插件的持久数据目录（更新后保留） | 放 `node_modules`、缓存、跨版本要保留的状态 |
| `${CLAUDE_PROJECT_DIR}` | 项目根目录 | 引用项目里的脚本/配置 |

**类比：活页夹里写「本夹位置」，而不是抄死页码。** 想象一份会被反复重新装订的活页夹，你要在里头写「去附录那页」。如果你写死「第 87 页」，一重新装订页码全乱；聪明的做法是写「本夹的附录」这种**相对本夹的占位说法**。`${CLAUDE_PLUGIN_ROOT}` 就是这个「本店地址」占位符——**不管插件被装到哪、更新到第几版，它永远指向当前这一版的根目录**。

所以插件里引用自带脚本，标准写法长这样（注意双引号，防路径带空格）：

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "\"${CLAUDE_PLUGIN_ROOT}\"/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

这里有个**官方反复强调的硬规矩**，值得贴脑门上：

> 当 plugin 更新时，此路径会更改。前一个版本的目录在更新后约七天内保留在磁盘上以进行清理，但应将其视为临时的，不要在此处写入状态。

翻译一下：**`${CLAUDE_PLUGIN_ROOT}` 是临时的，别往里写要长期保留的东西**（比如装好的依赖、缓存）——那些该写进 `${CLAUDE_PLUGIN_DATA}`，因为数据目录跨版本保留。常见的翻车是把插件装的 `node_modules` 放进 ROOT，结果插件一更新依赖全没了，得重装。**自带的脚本用 ROOT，要保留的状态用 DATA**，这条分清就不出事。

还有个相关坑，官方在「插件缓存」那节专门讲：**已安装的插件不能引用自己目录外的文件**。你写 `../shared-utils` 这种往外跳的相对路径，安装后会失效——因为插件被装时是**整个复制进缓存**（`~/.claude/plugins/cache`）的，外面的文件根本没被复制过去。要跨插件共享文件，得用符号链接，这是进阶话题，新手先记住「别往插件外引用」。

> 💡 一句话总结：插件里引用路径**必须用 `${CLAUDE_PLUGIN_ROOT}` 这类变量，不能写死绝对路径**（每次更新都变）；自带脚本用 `ROOT`、要保留的状态用 `DATA`，且**绝不引用插件目录外的文件**。

---

## 05 动手：从零造一个能跑的插件，再本地测

讲了四节构造，该真上手了。下面带你**亲手造一个最小但完整的插件、本地加载、跑通它的 skill**——全程不依赖任何复杂环境，跟着抄就行。我们造一个 `my-greeter` 插件，带一个打招呼的 skill。

**第一步：用官方脚手架命令起一个插件骨架**

最省事的不是手动建目录，而是用官方的 `plugin init` 命令，它会把骨架搭好：

```bash
claude plugin init my-greeter --with skills
```

`--with skills` 表示顺便搭一个示例 skill 文件夹。这条命令会在 `~/.claude/skills/my-greeter/` 下建好 `.claude-plugin/plugin.json` 和一个启动用的 `SKILL.md`。

**预期**：终端提示插件骨架已创建，告诉你建在了 `~/.claude/skills/my-greeter/`。

> 这里有个官方设计的便利：**放在 `~/.claude/skills/` 下、带 `plugin.json` 清单的文件夹，下个会话会自动作为 `my-greeter@skills-dir` 加载，不用建市场、不用安装**。这种叫「skills 目录插件」，是开发自用插件最轻的路子。

**第二步：看看脚手架生成了什么**

直接列一下目录（注意用绝对路径）：

```bash
ls -R ~/.claude/skills/my-greeter
```

**预期**：你会看到 `.claude-plugin/plugin.json` 和 `skills/`（或一个 `SKILL.md`）。**确认 `plugin.json` 在 `.claude-plugin/` 里、skill 在外面**——这就是第 01 节那条铁律的实物。

**第三步：写一个属于自己的 skill**

打开（或新建）`~/.claude/skills/my-greeter/skills/hello/SKILL.md`，内容写成：

```markdown
---
description: 用热情的语气跟用户打招呼
---

# Hello Skill

热情地跟名叫 "$ARGUMENTS" 的用户打招呼,问问今天能帮上什么忙。语气友好、鼓励一点。
```

`$ARGUMENTS` 是占位符——它会捕获你在 skill 名后面打的任何文字。这是给 skill 传参的标准做法（第 26、27 篇讲过 skill，这里复用）。

**第四步：本地加载这个插件**

开发期不用建市场，**用 `--plugin-dir` 直接加载本地插件目录**，这是官方专给开发测试的标志：

```bash
claude --plugin-dir ~/.claude/skills/my-greeter
```

**预期**：Claude Code 正常启动。敲 `/help`，能在插件命名空间下看到你的 skill 列出来。

**第五步：喊起你的 skill，亲眼看它跑**

插件 skill 永远带命名空间，所以这么调（后面跟个名字当参数）：

```text
/my-greeter:hello Walter
```

**预期**：Claude 用一段热情的、带上「Walter」这个名字的话回应你。**看到它按你写的 description 和正文打了招呼，说明这个插件不光建对了、它带的 skill 真能用了。**

**第六步：改了 skill，热重载看效果**

把 `SKILL.md` 的正文改两句（比如让它用中文加 emoji），然后**不用重启**，在 Claude Code 里敲：

```text
/reload-plugins
```

**预期**：重新跑一遍 `/my-greeter:hello Walter`，能看到改动生效了。

> ⚠️ 一个官方写明的区别：**改 `SKILL.md` 当前会话立即生效；但改 hooks、`.mcp.json`、`agents/` 这些，得 `/reload-plugins` 或重启才生效**。别改了 hook 发现没反应就以为写错了，先 reload 一下。

跑通这六步，你就把「造骨架 → 写组件 → 本地加载 → 调用 → 热重载」这条**插件开发的完整内循环**亲手走了一遍。**以后造任何插件，本质都是这套流程，只是组件更多。**

> 💡 一句话总结：`claude plugin init` 起骨架、`--plugin-dir` 本地加载、`/reload-plugins` 热重载——**这三条命令是插件开发的内循环**；放进 `~/.claude/skills/` 的还能免市场免安装自动加载。

---

## 06 把插件发出去：建一个市场

自用的插件，放 `~/.claude/skills/` 或 `--plugin-dir` 就够了。但**要发给团队、发给社区，得走「市场（marketplace）」这一层**——第 24 篇你是市场的「消费者」，这一节当一回市场的「店主」。

先理清两个**长得像、极易混的概念**，官方专门用 Note 框区分过：

| 概念 | 是什么 | 在哪定义 |
|------|--------|---------|
| **市场源（marketplace source）** | 去哪拿那本「商品目录」（`marketplace.json`） | 用户 `/plugin marketplace add` 时，或设置里 |
| **插件源（plugin source）** | 目录里每个插件本体从哪拿 | `marketplace.json` 里每个插件条目的 `source` 字段 |

**类比：商场 vs 商场里每件商品的供货地。** 「市场源」是这家商场开在哪、目录册从哪取；「插件源」是目录册里**每件商品**实际从哪个工厂发货。两者完全可以不在一处——商场目录可以挂在 A 仓库，里头某件商品从 B 工厂直发。

市场的核心是一个文件：**仓库根目录下的 `.claude-plugin/marketplace.json`**。它声明市场名、所有者、和插件清单。最小的一个长这样：

```json
{
  "name": "my-plugins",
  "owner": { "name": "Your Name" },
  "plugins": [
    {
      "name": "my-greeter",
      "source": "./plugins/my-greeter",
      "description": "A friendly greeting plugin"
    }
  ]
}
```

每个插件条目至少要 `name` 和 `source`（从哪拿这个插件）。`source` 支持好几种来源，这是市场最实用的部分：

| 插件源类型 | 怎么写 | 适合 |
|-----------|--------|------|
| **相对路径** | `"./plugins/my-greeter"` | 插件就在市场同一个仓库里（最常见） |
| **github** | `{ "source": "github", "repo": "owner/repo" }` | 插件在另一个 GitHub 仓库 |
| **git-subdir** | 给 `url` + `path` | 插件在某个大仓库（monorepo，多项目合一仓库）的子目录，稀疏克隆省带宽 |
| **npm** | `{ "source": "npm", "package": "@org/plugin" }` | 作为 npm 包发布的插件 |

**动手：把第 05 节那个插件，装进一个本地市场试试。** 假设你按上面的结构建好了 `my-marketplace/`（里头 `.claude-plugin/marketplace.json` + `plugins/my-greeter/`），在 Claude Code 里：

```text
/plugin marketplace add ./my-marketplace
/plugin install my-greeter@my-plugins
```

**预期**：第一条提示市场添加成功；第二条把插件装上。然后 `/my-greeter:hello` 就能调了——**这条链路跟第 24 篇你装别人插件时一模一样，只不过这次货是你自己的**。

发布前**一定先验证**，官方给了专门命令：

```bash
claude plugin validate ./my-marketplace
```

**预期**：检查 `marketplace.json` 的 schema、有没有重复插件名、源路径有没有非法的 `..`、版本对不对得上。**指向市场目录时它只查 `marketplace.json`；要连 skill/agent 的 frontmatter 一起查，得指向具体插件目录**（`claude plugin validate ./my-marketplace/plugins/my-greeter`）。

真要发给团队，把这个市场仓库推到 GitHub，同事一句 `/plugin marketplace add owner/repo` 就能加。**想自动让团队装**，可以在项目的 `.claude/settings.json` 里写 `extraKnownMarketplaces`，同事信任项目目录时会被提示安装——这块属于团队配置，新手先知道有这条路即可。

> 💡 一句话总结：发布走市场——根目录建 `.claude-plugin/marketplace.json`，列出插件和它们的 `source`（相对路径/github/npm 等）；**分清「市场源」（目录从哪来）和「插件源」（每个插件从哪来）**；发布前 `claude plugin validate`。

---

## 07 一组对照：自用、本地测、正式发布，三条路别走错

造插件的人最常犯的不是写错代码，是**用错了「分发方式」**——明明只是自己试，却去吭哧建市场；或者要发团队了，还在用只对自己生效的 `--plugin-dir`。这一节一张表把三条路钉清。

| 你的处境 | 该用 | 命令 | 特点 |
|---------|------|------|------|
| **纯自用、想随手改随手用** | skills 目录插件 | 放进 `~/.claude/skills/<名>/`（带 `plugin.json`） | 自动加载，免市场免安装，改 SKILL.md 立即生效 |
| **开发中、反复测一个本地包** | `--plugin-dir` | `claude --plugin-dir ./my-plugin` | 直接加载，不入库；可多次指定加载多个；`.zip` 也行 |
| **发给团队/社区** | 市场 | `marketplace.json` + `/plugin install` | 能版本管理、能自动更新、能共享 |

几个用起来才知道的细节：

**`--plugin-dir` 是开发利器，但只管当前这一次会话。** 你这次启动加载了，关掉再开就没了——它不写进任何配置。**好处是干净**：测完插件，关掉会话就一干二净，不留痕迹。开发时一个顺手的做法是一直挂着一个 `--plugin-dir` 的会话，改完 `/reload-plugins`，比每次重装市场快太多。官方还有个贴心设计：**`--plugin-dir` 的本地副本，会临时盖过同名的已装市场插件**——所以你能拿本地改动直接覆盖测试一个线上插件，不用先卸载。

**skills 目录插件有个范围陷阱。** 放 `~/.claude/skills/`（个人级）的，每个项目都能用；但放项目 `.claude/skills/` 的（项目级），官方明确警告：**它只从你启动 Claude Code 的那个目录加载，不会像普通 skill 那样往仓库根目录上找**。所以从子目录启动会漏掉根目录的插件——**要么从仓库根启动，要么 `/reload-plugins`**。

**别一上来就建市场。** 这是过度工程的典型。官方的建议很实在：**先用 `.claude/` 散装或 `--plugin-dir` 快速迭代，准备好共享了再转成插件、建市场**。一个插件通常要在 `--plugin-dir` 下改十几轮才稳定，这期间建市场纯属给自己找麻烦——每改一次还得提交、刷新市场。

> 💡 一句话总结：三条路别走错——**自用塞 `~/.claude/skills/`、开发用 `--plugin-dir`、发布才建市场**；`--plugin-dir` 只活一次会话（干净），发布前先在前两条路上把插件迭代稳。

---

## 08 发布绕不开的两个坑：版本管理与依赖

插件能跑、能装，不代表能「发好」。**版本管理和依赖**是发布时两个最容易翻车的地方，官方都用警告框圈过，这一节专门填。

### 坑一：设了 version，推新提交却没人更新

这是**最反直觉、最坑人**的一个。Claude Code 怎么判断「插件有没有新版本」？它按这个顺序取版本号：

1. `plugin.json` 里的 `version`
2. 市场条目里的 `version`
3. 都没有的话，用 git 提交的 SHA

关键就在这儿。官方警告说得很重：

> 设置 `version` 会固定 plugin。如果 `plugin.json` 声明 `"version": "1.0.0"`，推送新提交而不改变该字符串对现有用户没有任何作用，因为 Claude Code 看到相同的版本并保留缓存副本。

翻译成大白话：**你一旦写死了 `"version": "1.0.0"`，光往仓库推新代码是没用的——用户那边版本号没变，`/plugin update` 会回你「已是最新」，缓存压根不刷新。** 你必须每次发版都手动把版本号往上提（`1.0.1`、`1.1.0`……）。

所以官方给了两种策略，二选一，别混：

| 策略 | 怎么做 | 更新行为 | 适合 |
|------|--------|---------|------|
| **显式版本** | `plugin.json` 里设 `version`，每次发版手动提 | 用户只在你提版本号时收到更新 | 有稳定发布节奏的正式插件 |
| **提交 SHA 版本** | **不写** `version`,git 托管 | 每推一个新提交都算新版本，用户自动拿到 | 内部、团队、快速迭代的插件 |

**还有个加倍坑的：别在 `plugin.json` 和市场条目里都写 `version`。** 官方明确——`plugin.json` 的值会**无声地**盖过市场条目的，你在 `marketplace.json` 里改了版本可能根本不生效，因为被那个陈旧的 `plugin.json` 版本号压住了。

显式版本这个坑很容易栽：一个团队内部插件设了 `"version": "1.0.0"`，改了好几次代码推上去，同事一直反馈「怎么没更新」。你很容易以为是市场没刷新，折腾半天才反应过来——**版本号没动，Claude Code 当然认为没变化**。内部插件一律不写 `version`、让它用 SHA，推一次同事更新一次，就清净了。**结论：快速迭代的内部插件别设 version，稳定发布的正式插件才设、且每次记得提。**

### 坑二：依赖怎么声明

如果你的插件 A 必须有插件 B 才能工作，用 `dependencies` 字段声明：

```json
{
  "name": "my-plugin",
  "dependencies": [
    "helper-lib",
    { "name": "secrets-vault", "version": "~2.1.0" }
  ]
}
```

声明后，**装/启用你的插件时，Claude Code 会自动把依赖也装上/启上**。版本可以用 semver（语义版本范围）约束（像 `~2.1.0`），免得依赖出个大版本把你的插件搞崩。卸载时，`claude plugin uninstall --prune` 能顺手清掉那些「只为满足依赖而自动装、现在没人要」的插件；**你手动直接装的插件永远不会被 prune 碰**。

**类比：配料表里标「需另购的配件」。** 一份家具说明书末尾写着「本款需另购 M4 螺丝包（2.1 规格）」——你照单备齐配件，家具才装得起来。`dependencies` 就是插件的这张配件清单：**它把「我需要谁、要哪个版本」写明，Claude Code 照单自动备齐**。

> 💡 一句话总结：发布两个坑——**设了 `version` 就必须每次发版手动提，否则用户收不到更新**（快速迭代干脆别设、用 SHA）；依赖用 `dependencies` 声明、可加 semver 约束，装你的插件时会自动连依赖一起装。

---

## 09 小结

这一篇把插件从「会用」推进到「会造、会发」——**它本质就是一盒乐高：一本说明书（`plugin.json`）定身份，几格零件盒（组件文件夹）装能力，整盒打包就能发给任何人**。

把核心要点串起来回顾：

| 你要搞清的事 | 关键点 |
|------------|--------|
| 目录骨架 | `.claude-plugin/` 里**只放 `plugin.json`**，其余组件全摆根目录 |
| 清单字段 | 唯一必填 `name`（定命名空间）；组件路径分「替换」（agents/commands）和「追加」（skills） |
| 能装哪些组件 | skill/command/agent/hook/MCP/LSP/monitor 七类；**插件里的 agent 被削权**（无 hook/MCP/权限模式） |
| 路径怎么写 | 必须用 `${CLAUDE_PLUGIN_ROOT}` 等变量，**不写死、不引用目录外文件**；状态放 `DATA` 不放 `ROOT` |
| 开发内循环 | `plugin init` 起骨架 → `--plugin-dir` 加载 → `/reload-plugins` 热重载 |
| 发布 | 建 `marketplace.json`，分清市场源/插件源，`claude plugin validate` 验证 |
| 两个坑 | 设了 `version` 必须每次提；依赖用 `dependencies` 声明 |

**你现在应该能：** 看懂任何一个插件的目录在干嘛、自己从零搭一个能跑的插件骨架、用 `--plugin-dir` 本地测、再建个市场把它发给团队；也清楚 `${CLAUDE_PLUGIN_ROOT}` 为什么不能写死、`version` 那个「不提就不更新」的坑怎么躲。**第 24 篇你学会了消费别人的插件，这一篇你成了能生产插件的人——把自己那套攒了好久的配置打成一个包发出去，从此不再是只能抄别人的。**

到这里，Claude Code 的「扩展 + 配置 + 打包分发」这条线就齐全了。从第 18 篇的 `CLAUDE.md`、第 22 篇的 MCP、第 23 篇的 subagent，到第 24、38 这两篇的插件——**你不光会用这些工具，还会把它们组装、打包、交付给别人**。

---

下一篇 **39「实战入门」**——前面三十多篇全是「一个个功能拆开讲」，你手里的零件已经堆成山了。下一篇不再讲新功能，而是**把这些零件第一次串成一条完整的实战路径**：拿一个真实的小需求，从开工到交付走一遍，让你亲眼看见「学过的东西怎么拧成一股绳干活」。想想看——你现在会的每一招，单独使都顺手，**但真打一仗时，该先抽哪把、再接哪把？**
