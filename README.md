# Codex 中文教程与 AI 编程指南（含 Claude Code）

**简体中文** | [English](./README.en.md)

> 📘 **92 篇 · 约 52 万字** 精修中文教程，主力推荐 **Codex 39 篇**，同时保留 **Claude Code 53 篇**。适合 0 基础学习，从安装入门一直走到工程实战。

[![website](https://img.shields.io/badge/在线阅读-coding.stormzhang.ai-a6e3a1)](https://coding.stormzhang.ai)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Articles](https://img.shields.io/badge/articles-92-blue.svg)](#目录)

**📖 在线阅读（暗色终端风、体验更佳）→ <https://coding.stormzhang.ai>**

**主力推荐：从 0 基础开始学习 [Codex 中文教程与中文指南](https://coding.stormzhang.ai/codex/)**

同时提供：[Claude Code 中文教程](https://coding.stormzhang.ai/claude-code/)

![AI 编程指南](og.png)

## 这教程跟其他教程的差异

- **以官方文档为事实来源**：所有功能 / 命令 / 默认行为对照 [Codex 官方](https://developers.openai.com/codex) 和 [Claude Code 官方](https://code.claude.com/docs/zh-CN) 核实，不抄第三方猜测、不靠传言。
- **面向小白做大量易懂化改写**：每个新概念三段式（场景引入 + 生活化类比 + 实际场景）；不熟命令行也能跟上。
- **有真实经验印记**：每篇 3+ 处第一人称踩坑/判断（带具体细节、真实数字），不写「我觉得」式空话。
- **可照跑、可自验**：每个动手环节给完整命令 + 预期输出，让你边读边动手即时反馈。
- **暗色工程风原创配图**：81 张 SVG/PNG 配图，统一暗色风格、不堆字、节点 ≤ 10。

## 目录

### Claude Code 篇（53 篇）
从「是什么 / 安装」到代理循环、MCP、子代理、Skill、Hooks、Agent SDK、GitHub Actions，再到最佳实践 / 反模式 / FAQ / 术语表。

→ 从这里开始：[coding.stormzhang.ai](https://coding.stormzhang.ai)

<details open>
<summary><b>完整 53 篇目录</b></summary>

- [01 · Claude Code 简介](https://coding.stormzhang.ai/claude-code/01-what-is-claude-code)
- [02 · 安装与使用](https://coding.stormzhang.ai/claude-code/02-install)
- [03 · Claude Code 如何工作](https://coding.stormzhang.ai/claude-code/03-how-it-works)
- [04 · API 配置：订阅登录还是 API key，怎么选、怎么切](https://coding.stormzhang.ai/claude-code/04-api-config)
- [05 · 接入第三方 / 国产模型](https://coding.stormzhang.ai/claude-code/05-third-party-models)
- [06 · Coding Plan：订阅套餐与计费](https://coding.stormzhang.ai/claude-code/06-coding-plan)
- [07 · 第一次使用：跑通第一个例子](https://coding.stormzhang.ai/claude-code/07-first-run)
- [08 · VS Code 集成](https://coding.stormzhang.ai/claude-code/08-vscode)
- [09 · JetBrains 集成](https://coding.stormzhang.ai/claude-code/09-jetbrains)
- [10 · 桌面 app（Desktop）](https://coding.stormzhang.ai/claude-code/10-desktop)
- [11 · 网页版与云端：把 Claude Code 装进浏览器和手机](https://coding.stormzhang.ai/claude-code/11-web-and-cloud)
- [12 · 项目初始化：用 /init 一键生成 CLAUDE.md](https://coding.stormzhang.ai/claude-code/12-project-init)
- [13 · 项目结构：Claude Code 在你项目里都放了什么](https://coding.stormzhang.ai/claude-code/13-project-structure)
- [14 · 交互界面与快捷键：把手放对地方](https://coding.stormzhang.ai/claude-code/14-interface-and-shortcuts)
- [15 · 怎么提问和给指令：把话说到 Claude 心坎里](https://coding.stormzhang.ai/claude-code/15-prompting)
- [16 · 四个最常用的活儿：探索代码库、修 bug、重构、写测试](https://coding.stormzhang.ai/claude-code/16-common-workflows)
- [17 · 图片与多模态：贴张截图，它就懂了](https://coding.stormzhang.ai/claude-code/17-images-multimodal)
- [18 · CLAUDE.md 使用指南：把项目规矩写进它的记忆](https://coding.stormzhang.ai/claude-code/18-claude-md-guide)
- [19 · 上下文管理：别让它「失忆」也别烧爆 token](https://coding.stormzhang.ai/claude-code/19-context-management)
- [20 · 权限配置：放多松、收多紧，你说了算](https://coding.stormzhang.ai/claude-code/20-permissions)
- [21 · 安全与风险边界：到底该不该信任 AI 碰你的代码](https://coding.stormzhang.ai/claude-code/21-security)
- [22 · MCP：给 Claude 接上外部世界](https://coding.stormzhang.ai/claude-code/22-mcp)
- [23 · 子代理（Subagents）：把活儿外包出去，别什么都自己扛](https://coding.stormzhang.ai/claude-code/23-subagents)
- [24 · 插件（Plugins）：把一堆零碎配置一键打包](https://coding.stormzhang.ai/claude-code/24-plugins)
- [25 · 记忆系统（memory）：让它跨会话记住你](https://coding.stormzhang.ai/claude-code/25-memory)
- [26 · Agent Skills：给 Claude 装一身随叫随到的专项本事](https://coding.stormzhang.ai/claude-code/26-agent-skills)
- [27 · Skills 使用实例：装一个、喊一声、看它干活](https://coding.stormzhang.ai/claude-code/27-skills-in-practice)
- [28 · skill-creator 使用：用一个 skill 造你自己的 skill](https://coding.stormzhang.ai/claude-code/28-skill-creator)
- [29 · Agent teams 智能体团队：多会话协作](https://coding.stormzhang.ai/claude-code/29-agent-teams)
- [30 · 功能怎么选：CLAUDE.md vs Skill vs Hook vs MCP vs Subagent](https://coding.stormzhang.ai/claude-code/30-choosing-features)
- [31 · settings.json：用户级 / 项目级配置](https://coding.stormzhang.ai/claude-code/31-settings-json)
- [32 · 输出样式（Output Styles）：换一档「节目」，不换主持人](https://coding.stormzhang.ai/claude-code/32-output-styles)
- [33 · 钩子（Hooks）：在固定时机自动扣扳机](https://coding.stormzhang.ai/claude-code/33-hooks)
- [34 · CLI 参考手册：命令与全部标志](https://coding.stormzhang.ai/claude-code/34-cli-reference)
- [35 · 控制与模式：开会话时手里那块「调音台」](https://coding.stormzhang.ai/claude-code/35-modes-and-control)
- [36 · 斜杠命令（Slash Commands）：一个 `/` 调出 Claude 的所有快捷动作](https://coding.stormzhang.ai/claude-code/36-slash-commands)
- [37 · 检查点（Checkpoints）：随时能倒带的安全网](https://coding.stormzhang.ai/claude-code/37-checkpoints)
- [38 · 插件参考手册：把自己那套配置，打成一个能发出去的包](https://coding.stormzhang.ai/claude-code/38-plugins-reference)
- [39 · 实战入门：拿一个真需求，从开工到交付走一整趟](https://coding.stormzhang.ai/claude-code/39-getting-started-practice)
- [40 · Chrome：让它操作浏览器](https://coding.stormzhang.ai/claude-code/40-chrome)
- [41 · 并行任务：让几个 Claude 同时开工，而不是排队](https://coding.stormzhang.ai/claude-code/41-parallel-tasks)
- [42 · 环境变量：藏在背后那排「总开关」](https://coding.stormzhang.ai/claude-code/42-env-vars)
- [43 · Git 工作流：让 Claude 当你的 git 副手](https://coding.stormzhang.ai/claude-code/43-git-workflow)
- [44 · GitHub Actions：在 PR 里 @ 一下，让 Claude 自己干活](https://coding.stormzhang.ai/claude-code/44-github-actions)
- [45 · Agent SDK：把 Claude Code 的能力搬进你自己的程序](https://coding.stormzhang.ai/claude-code/45-agent-sdk)
- [46 · 开发配置：把 Claude 干活的「工作环境」调顺](https://coding.stormzhang.ai/claude-code/46-dev-config)
- [47 · Voice 语音模式：把提示词说出来，而不是打出来](https://coding.stormzhang.ai/claude-code/47-voice)
- [48 · 综合实战：从零到上线，把所学串成一条线](https://coding.stormzhang.ai/claude-code/48-capstone-project)
- [49 · 最佳实践：把零散的好习惯，攒成一套能照着做的心法](https://coding.stormzhang.ai/claude-code/49-best-practices)
- [50 · 反模式：常见的错误用法](https://coding.stormzhang.ai/claude-code/50-anti-patterns)
- [51 · 常见问题排查（FAQ / Troubleshooting）](https://coding.stormzhang.ai/claude-code/51-troubleshooting)
- [52 · 术语表（小白友好）：把这一路的「黑话」一次性翻译成人话](https://coding.stormzhang.ai/claude-code/52-glossary)
- [53 · 制作视频（Remotion）〔选读〕](https://coding.stormzhang.ai/claude-code/53-remotion-video)

</details>

### Codex 篇（39 篇）
四种入口、AGENTS.md、沙箱审批、config.toml、Chronicle 记忆、Worktrees、从 Claude Code 迁移等。

→ 从这里开始：[coding.stormzhang.ai](https://coding.stormzhang.ai)

<details open>
<summary><b>完整 39 篇目录</b></summary>

- [01 · 认识 Codex 与四种入口](https://coding.stormzhang.ai/codex/01-what-is-codex)
- [02 · Codex 核心概念速览](https://coding.stormzhang.ai/codex/02-core-concepts)
- [03 · 安装与登录（Mac / Windows / Linux）](https://coding.stormzhang.ai/codex/03-install)
- [04 · 订阅与计费](https://coding.stormzhang.ai/codex/04-pricing)
- [05 · 接入 DeepSeek 等国产模型](https://coding.stormzhang.ai/codex/05-third-party-models)
- [06 · 跑通第一个任务](https://coding.stormzhang.ai/codex/06-first-task)
- [07 · 桌面 App 全景](https://coding.stormzhang.ai/codex/07-desktop-app)
- [08 · 命令行 CLI 上手](https://coding.stormzhang.ai/codex/08-cli)
- [09 · IDE 扩展（VS Code 等）](https://coding.stormzhang.ai/codex/09-ide)
- [10 · 云端 Codex Cloud：把活丢上云，喝着咖啡等结果](https://coding.stormzhang.ai/codex/10-cloud)
- [11 · 项目说明书 AGENTS.md：把规矩焊进 Codex 的开工流程](https://coding.stormzhang.ai/codex/11-agents-md)
- [12 · 斜杠命令与快捷键：会话里的「快捷操作面板」](https://coding.stormzhang.ai/codex/12-slash-commands)
- [13 · 提示词（Prompt）写法：把话说到 Codex 心坎里](https://coding.stormzhang.ai/codex/13-prompting)
- [14 · 四类日常工作流：探索、修 bug、重构、写测试](https://coding.stormzhang.ai/codex/14-workflows)
- [15 · 权限、沙箱与审批：放多松、收多紧，自己拧](https://coding.stormzhang.ai/codex/15-permissions)
- [16 · 安全与风险边界：到底该不该放手让它碰你的代码](https://coding.stormzhang.ai/codex/16-security)
- [17 · 电脑操控与浏览器（Computer Use）：让 Codex 长出手](https://coding.stormzhang.ai/codex/17-computer-use)
- [18 · config.toml 配置详解：一个文件管住所有旋钮](https://coding.stormzhang.ai/codex/18-config)
- [19 · 记忆系统（Memories 与 Chronicle）：让 Codex 跨会话记住你](https://coding.stormzhang.ai/codex/19-memory)
- [20 · 用 MCP 接外部工具：给 Codex 装上「外接口」](https://coding.stormzhang.ai/codex/20-mcp)
- [21 · 子代理（Subagents）：把活儿拆出去并行跑，但只有「你开口」它才拆](https://coding.stormzhang.ai/codex/21-subagents)
- [22 · Agent Skills 技能：把一套活儿打包，教会 Codex 自己接](https://coding.stormzhang.ai/codex/22-skills)
- [23 · 插件（Plugins）：一键装一整套能力，别再一个个手配](https://coding.stormzhang.ai/codex/23-plugins)
- [24 · 规则与钩子（Rules & Hooks）：给 Codex 装上「卡点」和「扳机」](https://coding.stormzhang.ai/codex/24-hooks)
- [25 · Worktrees 并行隔离：让几个 Codex 各干各的，互不打架](https://coding.stormzhang.ai/codex/25-worktrees)
- [26 · Git 与 GitHub 集成：让 Codex 在你的 PR 里当审查员](https://coding.stormzhang.ai/codex/26-git-github)
- [27 · 自动化与 CI/CD：让 Codex 在你不在的时候自己干活](https://coding.stormzhang.ai/codex/27-automation)
- [28 · 非交互模式 codex exec：把它塞进脚本和 CI 里跑](https://coding.stormzhang.ai/codex/28-noninteractive)
- [29 · Slack / Linear 与 SDK 集成：在别处召唤 Codex，把它嵌进你自己的产品](https://coding.stormzhang.ai/codex/29-integrations)
- [30 · 怎么选模型：同一句话，到底该派哪个模型去跑](https://coding.stormzhang.ai/codex/30-models)
- [31 · 进阶技巧与提速：拖慢你的不是模型，是你给的烂上下文](https://coding.stormzhang.ai/codex/31-speed)
- [32 · 从 Claude Code 迁移：旧地图换个工具，照样能找到家](https://coding.stormzhang.ai/codex/32-migrate-from-claude-code)
- [33 · Windows 使用要点：原生还是 WSL，到底怎么跑才省心](https://coding.stormzhang.ai/codex/33-windows)
- [34 · 综合实战：从零给一个 TODO 小工具加功能、提交一次](https://coding.stormzhang.ai/codex/34-capstone)
- [35 · 命令与配置速查表](https://coding.stormzhang.ai/codex/35-cheatsheet)
- [36 · 最佳实践：那些「正确的废话」之外，真正能落地的几条](https://coding.stormzhang.ai/codex/36-best-practices)
- [37 · 常见问题排查：装不上、登不了、不肯改文件，挨个拆](https://coding.stormzhang.ai/codex/37-faq)
- [38 · 术语表](https://coding.stormzhang.ai/codex/38-glossary)
- [39 · 企业管理与治理：一个人玩和一家公司用，是两件事](https://coding.stormzhang.ai/codex/39-enterprise)

</details>

## 怎么读

1. 挑一个工具（建议从 Codex 起步）。
2. 按编号顺序往下读，每篇 3-10 分钟。
3. **边读边动手** —— 每篇都有可照跑的命令 + 预期输出。
4. 学完一个章节就把它集成进你每天的开发流程。

## 常见疑问

| Q | A |
|---|---|
| **要付费吗？** | 教程本身 **完全免费**（MIT）。但 Claude Code / Codex 这两个工具本身需要订阅或 API 付费，各位按自己需求订阅就好 |
| **没用过命令行能学吗？** | 能。第一组「基础入门」专门照顾新手，从命令行基础带起 |
| **Claude Code 和 Codex 学哪个？** | 两个工具理念接近、各有所长。**新手建议从 Codex 起步**——门槛更低、对国内用户更友好；Claude Code 账号风控更严、封号风险更高，等熟悉了再上手更稳。Codex 篇有「从 Claude Code 迁移」横向对比，两边可无缝切换 |
| **教程会更新吗？** | 这两个工具迭代很快，重要变化会跟进。版本信息见 [Commits](https://github.com/stormzhang/ai-coding-guide/commits/main) |

## 贡献 / 反馈

- 发现错别字、过时信息、坏链接 → 直接[提 Issue](https://github.com/stormzhang/ai-coding-guide/issues/new)
- 想改进某句表述、补充踩坑案例 → 欢迎 PR
- 想加新主题、新工具 → 先开 Issue 讨论方向

## 状态

🟢 **稳定**：92 篇全部成稿，已完成四轮审核（事实核实 / 表达优化 / 16 子代理复核）。后续以增量小修小补为主，新工具上线时新增专篇。

---

如果这套教程帮你少踩坑、少绕弯，⭐ Star 一下让更多人看见。

## License

[MIT](LICENSE) © 2026 [stormzhang](https://github.com/stormzhang)
