# AI 编程（Claude Code + Codex）指南

> 📘 **92 篇精修中文教程**，带你从装好到熟练 —— **Claude Code 53 篇 + Codex 39 篇**，把命令行变成你最快的那只手。

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![GitHub stars](https://img.shields.io/github/stars/stormzhang/ai-coding-guide?style=social)](https://github.com/stormzhang/ai-coding-guide/stargazers)
[![Articles](https://img.shields.io/badge/articles-92-blue.svg)](#目录)
[![Built with VitePress](https://img.shields.io/badge/Built%20with-VitePress-646cff.svg)](https://vitepress.dev)

![AI 编程指南](og.png)

## 这教程跟其他教程的差异

- **以官方文档为事实来源**：所有功能 / 命令 / 默认行为对照 [Claude Code 官方](https://code.claude.com/docs/zh-CN) 和 [Codex 官方](https://developers.openai.com/codex) 核实，不抄第三方猜测、不靠传言。
- **面向小白做大量易懂化改写**：每个新概念三段式（场景引入 + 生活化类比 + 实际场景）；不熟命令行也能跟上。
- **有真实经验印记**：每篇 3+ 处第一人称踩坑/判断（带具体细节、真实数字），不写「我觉得」式空话。
- **可照跑、可自验**：每个动手环节给完整命令 + 预期输出，让你边读边动手即时反馈。
- **暗色工程风原创配图**：64 张 SVG/PNG 配图，统一暗色风格、不堆字、节点 ≤ 10。

## 目录

### Claude Code 篇（53 篇）
从「是什么 / 安装」到代理循环、MCP、子代理、Skill、Hooks、Agent SDK、GitHub Actions，再到最佳实践 / 反模式 / FAQ / 术语表。

→ 从这里开始：[`claude-code/01-what-is-claude-code.md`](claude-code/01-what-is-claude-code.md)

### Codex 篇（39 篇）
四种入口、AGENTS.md、沙箱审批、config.toml、Chronicle 记忆、Worktrees、从 Claude Code 迁移等。

→ 从这里开始：[`codex/01-what-is-codex.md`](codex/01-what-is-codex.md)

## 怎么读

1. 挑一个工具（建议从 Claude Code 起步）。
2. 按编号顺序往下读，每篇 3-10 分钟。
3. **边读边动手** —— 每篇都有可照跑的命令 + 预期输出。
4. 学完一个章节就把它集成进你每天的开发流程。

## 常见疑问

| Q | A |
|---|---|
| **要付费吗？** | 教程本身 **完全免费**（MIT）。但 Claude Code / Codex 这两个工具本身需要订阅或 API 付费（订阅最便宜，每月 $20 起，详见各篇「订阅与计费」） |
| **需要魔法上网吗？** | 安装、登录、使用都需要。教程里已标注每个工具的具体要求 |
| **没用过命令行能学吗？** | 能。第一组「基础入门」专门照顾新手，从命令行基础带起 |
| **Claude Code 和 Codex 学哪个？** | 看你常用什么模型。两个工具理念接近、各有所长。推荐先从 Claude Code 起步，再读 Codex 篇的「从 Claude Code 迁移」横向对比 |
| **教程会更新吗？** | 这两个工具迭代很快，重要变化会跟进。版本信息见 [Commits](https://github.com/stormzhang/ai-coding-guide/commits/main) |

## 技术栈

教程站基于 **VitePress** 搭建，**Catppuccin Mocha** 暗色主题（仿 Claude Code CLI 会话窗口风格），代码块 Shiki 高亮，配图 SVG → @2x PNG，搜索 / mermaid / OG 卡片开箱即用。

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
