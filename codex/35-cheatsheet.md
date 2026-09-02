---
seoTitle: "Codex 命令与 config.toml 配置速查表"
description: "汇总 Codex CLI 参数、斜杠命令、权限模式、config.toml 字段和常用文件路径，适合日常使用时快速定位准确写法，并给出适合中文开发者直接照做的操作思路、检查方法与风险边界。"
published: "2026-06-12"
lastVerified: "2026-09-02"
author: stormzhang
officialSources:
  - https://developers.openai.com/codex/cli/reference
related:
  - ./38-glossary.md
  - ./36-best-practices.md
---
# 35 · 命令与配置速查表

> 📚 **系列导航**：上一篇〔[34 综合实战](34-capstone.md) 〕带你把前面所有零件串成一个完整项目跑了一遍，那是「合」。这一篇反过来——把散落在三十多篇里的命令、标志、配置键全部抽出来，压成一张你能贴在显示器边上的速查表。下一篇〔[36 最佳实践](36-best-practices.md) 〕再从「怎么用得对」收尾整个 Codex 篇。

说句丢人的事。

用 Codex 的头两个月，我电脑里躺着一个叫 `codex-备忘.txt` 的文件，里面歪歪扭扭记着「导出 JSON 是 `--json`」「写最后一条消息到文件是 `-o`」这类东西。问题是我记得乱，每次要用 `codex exec` 把结果写进文件，我先翻这个 txt，翻不到就翻 shell 历史 `history | grep codex`，再翻不到就开浏览器搜官方文档——一条命令的事，能折腾我五分钟。

更蠢的是配置。有回我想给某个项目临时关掉沙箱网络，记得有个 `sandbox_workspace_write` 开头的键，但后面接什么忘得一干二净，`network` ？`net_access` ？我对着 `config.toml` 瞎试，试到 Codex 启动报错，最后还是去翻 config-reference 才知道是 `sandbox_workspace_write.network_access` 。那一刻我决定：**与其每次现翻，不如一次性把高频命令和配置抠出来排成表，要哪个扫一眼就走。**

这一篇就是那张表的成品。它不讲原理（原理前面都讲过了），只干一件事——**让你查得快、抄得准、不用再开浏览器。**

**看完这一篇，你会拿到：**

- 一张覆盖安装登录、CLI 命令与标志、斜杠命令、`config.toml` 高频配置项的速查表
- 权限沙箱档位、模型与推理强度的对照表，照着填不踩坑
- MCP（Model Context Protocol，模型上下文协议）/ 子代理 / Skills 这些进阶能力的「关键入口」清单，知道从哪条命令切进去
- 一个能跑通验证的小例子，确认你查到的命令真在你本地生效

> ⚠️ 命令、标志、配置键以官方文档为准，**会随版本变化**；模型名、默认值随版本与你的账号而变，**一律以你本地 `codex --help` 和 `config.toml` 实际行为为准**；通过 `codex --version` 确认当前版本。下面标了「实验性」的，开头会注明，用之前留个心眼。

---

## 01 安装与登录

第一次装、换机器、CI 里重新拉起，最常用就这几条。**类比：这是 Codex 的「开机三件套」——装上、登进去、确认登成功，缺一步它都不干活。**

国内网络下，安装脚本和登录走 OAuth 都可能需要**魔法上网**，否则容易卡在下载或回调那一步。

| 目的 | 命令 | 平台 / 备注 |
| --- | --- | --- |
| 标准安装（脚本） | `curl -fsSL https://chatgpt.com/codex/install.sh \| sh` | macOS / Linux |
| 标准安装（脚本） | `powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 \| iex"` | Windows |
| 用 npm 装 | `npm install -g @openai/codex` | 全平台，需 Node |
| 用 Homebrew 装 | `brew install --cask codex` | macOS |
| 更新到最新版 | `codex update` | 发行版支持自更新时可用 |
| 登录（浏览器 OAuth） | `codex login` | 默认方式，开浏览器登 ChatGPT |
| 登录（设备码） | `codex login --device-auth` | 无法开浏览器时用（如远程服务器） |
| 用 API Key 登录 | `printenv OPENAI_API_KEY \| codex login --with-api-key` | 从 stdin 读 key |
| 查登录状态 | `codex login status` | 已登录退出码为 `0`，适合脚本判断 |
| 退出登录 | `codex logout` | 清掉本地凭据 |
| 体检诊断 | `codex doctor` | 装好/出问题时跑一把，自检安装、配置、认证、Git 等 |

> 💡 一句话总结：装完先 `codex doctor` 跑一遍体检，再用 `codex login status` 确认登录态，比直接开干少走一半弯路。

---

## 02 codex CLI 常用命令与标志

这是全篇最核心的一块。**类比：`codex` 是主程序，后面跟的是「子命令」（去哪），再后面的 `--xxx` 是「标志」（怎么去）。** 先记子命令，再记标志，组合起来就是一条完整命令。

### 常用子命令

| 子命令 | 干什么 | 成熟度 |
| --- | --- | --- |
| `codex` | 启动交互式终端界面（TUI，Terminal User Interface），不带子命令时就是它 | 稳定 |
| `codex exec` | 非交互跑一次，跑完即退；短写 `codex e` | 稳定 |
| `codex resume` | 接着上一次的交互会话继续聊 | 稳定 |
| `codex fork` | 把某次会话「分叉」成新线程，原记录不动 | 稳定 |
| `codex archive` / `codex unarchive` | 归档 / 恢复某个存过的会话——只清出 resume 列表，不删记录 | 0.140.0 新增 |
| `codex delete` | **永久删除**某个会话（有确认 safeguard；`--force` 仅限按 UUID 删） | 0.140.0 新增 |
| `codex agents` | 打开 agent 会话仪表盘：搜索 / 打开 / 重命名 / 停止本地各任务 | 0.149.0 新增 |
| `codex queue` | 给某个已在跑的会话排队发一条消息 | 0.149.0 新增 |
| `codex remote-control pair` | 给运行中的远程控制守护进程生成一个短时手动配对码 | 实验性，0.143.0 新增 |
| `codex apply` | 把云端任务生成的 diff 应用到本地；短写 `codex a` | 稳定 |
| `codex mcp` | 管理 MCP 服务器（list / add / remove / login） | 实验性 |
| `codex features` | 列出功能开关并持久启用/禁用 | 稳定 |
| `codex completion` | 生成 shell 补全脚本 | 稳定 |

### 高频全局标志（跟在 `codex` 或多数子命令后）

| 标志 | 作用 | 取值 / 备注 |
| --- | --- | --- |
| `--model` / `-m` | 临时换模型 | 如 `-m gpt-5.6-terra` |
| `--image` / `-i` | 附带图片给首条提示 | 多张用逗号分隔或重复 `-i` |
| `--cd` / `-C` | 指定工作目录后再开干 | 接一个路径 |
| `--sandbox` / `-s` | 选沙箱档位 | `read-only` / `workspace-write` / `danger-full-access` |
| `--ask-for-approval` / `-a` | 选审批时机 | `untrusted` / `on-request` / `never` |
| `--approve-for-me` | 审批请求改交自动审查代理（不扩沙箱边界） | 0.147.0 新增，详见第 15 篇 |
| `--search` | 开实时联网搜索 | 把 `web_search` 切到 `live`（默认是 `cached`） |
| `--add-dir` | 额外给某目录写权限 | 可重复，比直接放开全盘安全 |
| `--profile` / `-p` | 套用某个配置 profile | 叠在基础配置之上 |
| `--config` / `-c` | 命令行临时改配置 | `-c key=value`，能解析成 TOML 就按 TOML |
| `--yolo` | 跳过一切审批和沙箱 | **危险**，只在隔离环境里用 |

### `codex exec`（非交互）专属高频标志

这是写脚本、跑 CI 最常用的一组。**类比：交互模式像你在饭店点菜，`codex exec` 像下单外卖——下了就走，结果送到你指定的地方。**

| 标志 | 作用 |
| --- | --- |
| `PROMPT` 写 `-` | 从 stdin 读提示词（如 `cat prompt.txt \| codex exec -`） |
| `--json` | 输出按行的 JSON（JSONL）事件流，方便 `jq` 解析 |
| `--output-last-message` / `-o` | 把最终回复**写到文件**（同时仍打印到 stdout） |
| `--output-schema` | 给个 JSON Schema，强制最终输出符合该结构 |
| `--skip-git-repo-check` | 允许在非 Git 目录里跑 |
| `--ephemeral` | 不在磁盘留会话记录 |
| `--full-auto` | **已删除**（0.147.0 起彻底移除，不再兼容）；新脚本改用 `--sandbox workspace-write` |
| `codex exec resume --last` | 接着最近一次 exec 会话继续 |
| `codex exec fork` | 从某次会话分叉出新会话，原记录不动（0.148.0 新增） |

> 💡 一句话总结：`-m` 换模型、`-s` 调沙箱、`-a` 调审批、`-o`/`--json` 收结果——把这四件事记牢，命令行八成场景就齐了。

---

## 03 斜杠命令（在 TUI 里输入）

斜杠命令只在交互界面里用——进了 `codex` 那个全屏界面，敲 `/` 弹出菜单。**类比：它们是会话里的「快捷按钮」，不用退出去敲命令，打个斜杠就地切。**

下面是我自己用得最勤的一批。完整清单以你本地 `/` 弹出的菜单为准，不同版本会有增减。

| 斜杠命令 | 干什么 |
| --- | --- |
| `/model` | 切当前模型（可选时连推理强度一起调） |
| `/status` | 看当前模型、审批策略、可写目录、剩余上下文 |
| `/compact` | 把长对话压成摘要，腾出上下文空间 |
| `/diff` | 看 Git diff，连没跟踪的新文件也显示 |
| `/permissions` | 中途调整 Codex 能不问就做哪些事 |
| `/review` | 让 Codex 审一遍你当前工作区的改动 |
| `/init` | 在当前目录生成 `AGENTS.md` 脚手架 |
| `/mcp` | 列出当前会话能调的 MCP 工具（加 `verbose` 看详情） |
| `/skills` | 浏览并选用本地 skill |
| `/plugins` | 打开插件浏览器：装 / 卸插件、按空格切启用（详见第 23 篇） |
| `/hooks` | 查看 / 信任 / 禁用生命周期钩子（详见第 24 篇） |
| `/agent` | 在已派生的子代理线程之间切换 |
| `/fast` | 开/关当前模型的 Fast 服务层（`/fast on`/`off`/`status`） |
| `/new` | 同一 CLI 会话里开一段全新对话（0.146.0 起可顺带命名，如 `/new bug bash`） |
| `/clear` | 清屏并开新对话（同样可顺带命名） |
| `/import` | 从 Claude Code / Cursor 导入设置与最近对话（0.140.0 新增，0.145.0 起支持 Cursor） |
| `/export` | 把当前对话导出成 Markdown，到剪贴板或新文件（0.148.0 新增） |
| `/usage` | 看账号 token 用量（`daily` / `weekly` / `cumulative`），可兑换限额重置（0.140.0 新增） |
| `/delete` | 永久删除当前会话并退出（0.140.0 新增） |
| `/cd` `/pwd` `/cwd` | 查看 / 切换会话的工作目录（0.149.0 新增） |
| `/quit` 或 `/exit` | 退出 CLI |

提醒一句：`/fast` 是「模型目录（catalog）驱动」的——当前模型如果不提供 Fast 层，菜单里压根不出现 `/fast` ，别以为是 bug 。

> 💡 一句话总结：会话里最高频的就四个——`/model` 换脑子、`/status` 看现状、`/compact` 清场子、`/diff` 验成果，先把这四个练成肌肉记忆。

---

## 04 config.toml 常用配置项

配置文件在 `~/.codex/config.toml` （TOML 格式），是 Codex 的「长期记忆」。**类比：命令行标志是「这一趟怎么走」，`config.toml` 是「以后默认都这么走」。** 项目里还能放 `.codex/config.toml` 做项目级覆盖（需先信任该项目）。

下面只列高频项，全量见官方 config-reference 。

| 配置键 | 作用 | 取值示例 |
| --- | --- | --- |
| `model` | 默认模型 | `"gpt-5.6"` |
| `model_reasoning_effort` | 推理强度（可选值随模型变化） | 常见：`none` / `minimal` / `low` / `medium` / `high` / `xhigh` |
| `model_reasoning_summary` | 推理摘要详细度 | `auto` / `concise` / `detailed` / `none` |
| `service_tier` | 服务层级 | `flex` / `fast`（提速相关，搭配 `/fast`） |
| `sandbox_mode` | 沙箱档位 | `read-only` / `workspace-write` / `danger-full-access` |
| `sandbox_workspace_write.network_access` | 工作区写模式下是否放开联网 | `true` / `false` |
| `sandbox_workspace_write.writable_roots` | 额外可写目录 | `["/path/a", "/path/b"]` |
| `approval_policy` | 审批策略 | `untrusted` / `on-request` / `never` |
| `web_search` | 联网搜索模式 | `disabled` / `cached` / `indexed` / `live`（默认 `cached`） |
| `review_model` | `/review` 用的模型 | 不填则用当前会话模型 |
| `model_instructions_file` | 用某文件替代内置指令 | 一个路径 |

一个最小可用的 `config.toml` 长这样，复制进去改改就能用：

```toml
model = "gpt-5.6"
model_reasoning_effort = "medium"
sandbox_mode = "workspace-write"
approval_policy = "on-request"

[sandbox_workspace_write]
network_access = false
```

> 💡 一句话总结：`config.toml` 里把 `model`、`model_reasoning_effort`、`sandbox_mode`、`approval_policy` 四个键定好，就等于给 Codex 定了「默认人格」，剩下临时调用标志去覆盖。

---

## 05 权限 / 沙箱档位

沙箱决定 Codex 能动你机器到什么程度，审批决定它动手前问不问你。这俩是组合拳。**类比：沙箱是「实习生能进哪几个房间」，审批是「他动手前要不要先喊你一声」。**

| 沙箱档位（`--sandbox` / `sandbox_mode`） | 能干什么 | 适合场景 |
| --- | --- | --- |
| `read-only` | 只读，不能改文件 | 让它先看、先分析，不动代码 |
| `workspace-write` | 能改工作区内文件 | 日常本地开发的甜区 |
| `danger-full-access` | 全盘读写、放开网络 | **只在隔离容器 / CI runner 里用** |

| 审批时机（`--ask-for-approval` / `approval_policy`） | 含义 |
| --- | --- |
| `untrusted` | 只对它认为可信的命令放行，其余都问 |
| `on-request` | 交互式跑的推荐值，需要时才暂停问你 |
| `never` | 从不问，非交互/CI 跑用这个 |

官方给本地低摩擦干活的推荐组合就一句：

```bash
codex --sandbox workspace-write --ask-for-approval on-request
```

补充两个我踩过的点：想给 Codex 多开一个可写目录，**别图省事直接上 `danger-full-access`，用 `--add-dir` 精准放行那一个目录**就够；`--full-auto` 这个老标志已在 0.147.0 彻底删除，老脚本一律改成 `--sandbox workspace-write`。

> 💡 一句话总结：本地默认「`workspace-write` + `on-request`」，CI 用「指定沙箱 + `never`」，要放权先想能不能用 `--add-dir` 替代全开。

---

## 06 模型与推理强度对照

选模型是「派谁去」，调推理强度是「让他想多久再动手」，两个旋钮独立。**类比：模型是请的人，推理强度是给他的思考时间——简单活儿别让高手慢慢琢磨，硬骨头也别让新手草草交差。**

| 模型 | 定位 | 什么时候用 |
| --- | --- | --- |
| `gpt-5.6-sol` | 旗舰、默认，最强 | 复杂编程、重构、研究类硬活 |
| `gpt-5.6-terra` | 日常主力 | 日常开发，性能追平上代旗舰、更省 |
| `gpt-5.6-luna` | 轻量、快、省 | 杂活、批量任务、子代理 |
| `gpt-5.3-codex-spark` | 即时型研究预览（仅 ChatGPT Pro） | 求近乎秒回的实时迭代 |

`gpt-5.2` 和 `gpt-5.3-codex` 已弃用，`gpt-5.4` 和 `gpt-5.4-mini` 也已于 2026 年 8 月 31 日退役（ChatGPT 登录），都别再写进配置或 `--model` 里。

推理强度由 `model_reasoning_effort` 控制，**可选档位随模型变化**（哪几档能用，以 `/model` 面板和当前模型文档为准），常见档位：

| 取值 | 思考力度 | 典型场景 |
| --- | --- | --- |
| `none` | 完全不额外想（模型相关） | 纯指令执行 |
| `minimal` | 几乎不想，最快 | 改 typo、重命名、跑个命令 |
| `low` | 略想一下 | 小修小补 |
| `medium` | 默认甜区 | 绝大多数日常编程 |
| `high` | 深思熟虑 | 多文件改动、设计权衡 |
| `xhigh` | 顶格（模型相关） | 真·硬骨头，确定值得等 |

我自己的默认就是「`gpt-5.6` + `medium`」打天下，只在确定要啃硬骨头时手动提到 `high`。前面第 30 篇说过我犯的蠢——长期锁死 `xhigh` 改个 typo 等一分钟，**那种「最强焦虑」省下来的，比你以为的多得多。**

> 💡 一句话总结：日常「旗舰 + `medium`」，体力活降到 `minimal`/`low`，硬骨头才提 `high`/`xhigh`，弃用模型一个别碰。

---

## 07 MCP / 子代理 / Skills 关键入口

这三块都是进阶能力，原理前面各有专篇（第 20、21、22 篇）。这里只给「从哪条命令/配置切进去」的入口，省得你回头翻。**类比：这是三扇门的「门把手位置」——记住把手在哪，推门进去的细节回专篇看。**

| 能力 | 关键入口 | 说明 |
| --- | --- | --- |
| MCP（外接工具，像 USB 接口） | `codex mcp list` / `codex mcp add <name> ...` | 命令行管服务器；会话里 `/mcp` 看可用工具（实验性） |
| MCP（HTTP 服务器登录） | `codex mcp login <name>` | 仅支持 OAuth 的 streamable HTTP 服务器 |
| 配置 MCP 服务器 | `config.toml` 里 `[mcp_servers.<id>]` | `command`/`args`/`url` 等键定义一台服务器 |
| 子代理（并行干活） | `config.toml` 里 `[agents]` / `agents.<name>.*` | 并发上限键为 `agents.max_concurrent_threads_per_session`（旧名 `max_threads` 保留兼容）；会话里 `/agent` 切线程 |
| Skills（任务专用技能） | 会话里 `/skills` | 浏览并选用；`config.toml` 里用 `[[skills.config]]`（带 `path`/`enabled`）做启用覆盖 |

注意 `codex mcp` 整组目前标的是「实验性」，子命令和行为可能随版本变，用前以 `codex mcp --help` 为准。

> 💡 一句话总结：MCP 走 `codex mcp` 命令、子代理靠 `[agents]` 配置 + `/agent` 切换、Skills 用 `/skills` 选——记住这三个入口，进阶能力就不会「不知道从哪开始」。

---

## 08 动手：验证你查到的命令真在生效

速查表抄得再齐，不如亲手跑一条确认它在你本地真能用。下面这条最小例子，不依赖任何已有项目，三步走。

第一步，确认登录状态（不改任何东西，纯查）：

```bash
codex login status
```

预期输出：已登录会打印当前认证方式，**退出码为 `0`**；没登录会提示你去 `codex login`。

第二步，跑一条非交互命令，让它把结果写进文件——同时验证 `codex exec`、`-o` 和只读沙箱三件事：

```bash
codex exec --sandbox read-only -o /tmp/codex-check.txt "用一句话说明当前目录是不是一个 Git 仓库"
```

预期：终端打印出那句话，同时 `/tmp/codex-check.txt`（Windows 上换成你的临时目录）里也写下了同一句。打开文件能看到内容，就说明 `-o` 生效了。

第三步，确认你查的标志名没记错——直接问 CLI 自己：

```bash
codex exec --help
```

预期：列出 `codex exec` 支持的所有标志。**任何时候你拿不准某个标志存不存在、叫什么，`--help` 永远是比这张表更新、更准的那一份。**

> 💡 一句话总结：`login status` 验登录、`codex exec -o` 验命令真在干活、`--help` 验标志名没记错——这三条跑通，说明速查表对你本地是「活」的。

---

## 小结

这一篇没讲新概念，把前面三十多篇里的「手感」压成了七张表：

- **开机三件套**：安装、`codex login`、`codex login status` 确认，出问题先 `codex doctor`。
- **CLI 命令与标志**：子命令记「去哪」，标志记「怎么去」，`-m`/`-s`/`-a`/`-o`/`--json` 是高频五件套。
- **斜杠命令**：会话内 `/model`、`/status`、`/compact`、`/diff` 先练成肌肉记忆。
- **`config.toml`**：`model`、`model_reasoning_effort`、`sandbox_mode`、`approval_policy` 定好默认人格。
- **权限沙箱**：本地「`workspace-write` + `on-request`」，CI「指定沙箱 + `never`」，放权优先 `--add-dir`。
- **模型与强度**：旗舰 `gpt-5.6-sol` + `medium` 打天下，退役模型别碰。
- **进阶入口**：MCP 走 `codex mcp`、子代理靠 `[agents]`、Skills 用 `/skills`。

**你现在应该能**：丢掉那个歪歪扭扭的备忘 txt，要哪条命令、哪个配置键、哪个档位，扫一眼这页就走，拿不准时一句 `--help` 兜底，不用再开浏览器现翻文档。

---

下一篇〔[36 最佳实践](36-best-practices.md) 〕是整个 Codex 篇的收尾。速查表解决的是「记不住」，最佳实践解决的是「用得对」——同样一条 `codex exec`，有人拿它跑出一条顺滑的自动化流水线，有人拿它把代码库搅得一团乱。留个小思考：**你手上这些命令，哪几条是你天天在用、却从没想过有没有更稳的用法？** 下一篇咱们就把这些「天天用却没想透」的地方，一个个掰开。
