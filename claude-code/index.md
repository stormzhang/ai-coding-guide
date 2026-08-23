---
seoTitle: "Claude Code 中文教程：从安装入门到高级实战"
description: "面向中文初学者的 Claude Code 系统教程，覆盖安装登录、提示词、CLAUDE.md、权限、MCP、Skills、子代理、Hooks、Git 工作流与综合实战。"
published: "2026-08-23"
lastVerified: "2026-08-23"
author: stormzhang
pageClass: topic-hub
---

# Claude Code 中文教程

这是一套面向中文初学者的 Claude Code 系统教程。你可以从安装开始按顺序学习，也可以直接跳到权限、MCP、Skills、子代理或工程化实战；53 篇内容均以官方文档和真实操作为事实依据。

## 怎么选学习路线

- **第一次接触 AI 编程工具**：从第一组开始，按编号顺序学习，先跑通最小任务再碰高级配置。
- **已经会基础操作**：直接进入第三、四组，重点补齐工作流、权限、安全和扩展能力。
- **只想快速查资料**：从第七组的速查、排错和术语内容进入，再沿相关链接回到专篇。

> 💡 推荐起点：先读第 01 篇建立整体认识，再完成安装和第一个任务。遇到具体问题时，不必从头重学，按下面的分类直接跳转即可。

## 一 · 基础入门

先弄清 Claude Code 是什么，完成安装、认证与模型接入。

- **[01 · Claude Code 是什么？功能、原理与适用场景](./01-what-is-claude-code.md)** -- Claude Code 的产品定位、代理式工作方式、能做与不能做的事情，以及它和 ChatGPT、Copilot、Cursor 的核心区别
- **[02 · Claude Code 安装教程：Mac、Windows 与 Linux](./02-install.md)** -- Mac、Windows、Linux 和 WSL 的安装步骤、登录方法、网络要求与常见报错处理，让第一次使用命令行的读者也能完成安装
- **[03 · Claude Code 工作原理：代理循环与内置工具](./03-how-it-works.md)** -- Claude Code 的代理循环、上下文窗口、内置工具和权限确认机制，解释它如何从理解任务走到修改文件与验证结果
- **[04 · Claude Code API 配置：订阅登录与 API Key 怎么选](./04-api-config.md)** -- Claude 订阅登录、Anthropic API Key 与云厂商认证的差异、费用归属和切换方式，帮助你选对最合适的接入路径
- **[05 · Claude Code 接入 DeepSeek 等第三方模型](./05-third-party-models.md)** -- 通过兼容网关或第三方服务为 Claude Code 配置模型的基本原理、环境变量、验证方法和风险边界，避免把兼容接口当成官方能力

## 二 · 上手与项目

从第一次任务走到编辑器、桌面端、云端和项目规则。

- **[06 · Claude Code 订阅套餐与计费完整说明](./06-coding-plan.md)** -- Claude 各订阅套餐、API 计费与 Claude Code 使用额度的关系，并说明不同使用强度下怎么选方案、怎么看成本和避免意外消耗
- **[07 · Claude Code 第一次使用：从启动到完成任务](./07-first-run.md)** -- 首次进入项目、授权信任、发送任务、检查修改和验证结果的完整流程，用一个最小示例带你跑通 Claude Code 的第一次代理任务
- **[08 · Claude Code VS Code 集成与使用指南](./08-vscode.md)** -- Claude Code 在 VS Code 中的安装、启动、选区引用、差异查看与终端协作方式，帮助你把编辑器操作和代理任务连成一套工作流
- **[09 · Claude Code JetBrains 集成与使用指南](./09-jetbrains.md)** -- Claude Code 在 IntelliJ IDEA、PyCharm 等 JetBrains IDE 中的安装、上下文共享、差异查看和常用操作，说明插件与终端入口如何配合
- **[10 · Claude Code Desktop 桌面应用完整指南](./10-desktop.md)** -- Claude Code 桌面应用的安装、项目管理、会话操作和适用场景，并比较 Desktop、终端与网页版入口的能力边界和选择方式
- **[11 · Claude Code 网页版与云端使用指南](./11-web-and-cloud.md)** -- Claude Code Web 的仓库连接、云端任务、手机访问和 Deep Link 等能力，说明哪些工作适合放到云端、哪些仍应留在本地完成
- **[12 · Claude Code /init 教程：生成与维护 CLAUDE.md](./12-project-init.md)** -- 使用 /init 扫描项目并生成 CLAUDE.md 的流程、生成结果的检查方法和后续维护原则，让 Claude 快速理解项目结构与规则
- **[13 · Claude Code 项目目录与配置文件详解](./13-project-structure.md)** -- 项目中的 .claude 目录、CLAUDE.md、settings、commands、agents、skills 和 hooks 分别放什么，帮助你看懂配置层级与作用范围

## 三 · 核心交互与操作

掌握提示词、上下文、权限、安全和四类日常工作流。

- **[14 · Claude Code 交互界面与快捷键速查](./14-interface-and-shortcuts.md)** -- 交互界面的状态区、输入模式、历史记录、快捷键和 Shell 模式，帮助你减少鼠标操作并准确判断当前会话处于什么状态
- **[15 · Claude Code 提示词写法：任务、上下文与验收标准](./15-prompting.md)** -- 如何向 Claude Code 说明目标、提供上下文、约束改动范围和定义验收标准，并用可复用的提示词结构减少返工与误改
- **[16 · Claude Code 常见工作流：探索、修 Bug、重构与测试](./16-common-workflows.md)** -- 探索代码库、定位并修复 Bug、控制重构范围和补写测试四类高频任务的标准步骤、提示方式与验收方法
- **[17 · Claude Code 图片与多模态使用指南](./17-images-multimodal.md)** -- 向 Claude Code 提供截图、设计稿和错误界面的方式，以及图片上下文适合解决的问题、提示词写法与视觉还原时的验收要点
- **[18 · CLAUDE.md 完整指南：层级、写法与最佳实践](./18-claude-md-guide.md)** -- CLAUDE.md 的发现范围、层级优先级、推荐内容、反模式和维护方法，帮助你把稳定的项目规则写成 Claude 可执行的说明书
- **[19 · Claude Code 上下文管理：压缩、清理与 Token 控制](./19-context-management.md)** -- 上下文窗口如何消耗、何时使用 compact 或 clear、怎样减少无关材料和会话漂移，让长任务保持准确并控制 Token 成本
- **[20 · Claude Code 权限配置：模式、规则与审批](./20-permissions.md)** -- Claude Code 的权限模式、allow 与 deny 规则、工具审批和作用域配置，帮助你在效率与安全之间选择合适的开放程度
- **[21 · Claude Code 安全指南：风险边界与防护清单](./21-security.md)** -- 提示词注入、恶意仓库、命令执行、敏感信息和供应链风险，并给出审查权限、隔离环境与检查输出的实用防护原则

## 四 · 高级功能扩展

学会用 MCP、子代理、插件、记忆、Skills 和 Agent Teams 扩展能力。

- **[22 · Claude Code MCP 教程：连接外部工具与服务](./22-mcp.md)** -- MCP 的工作原理、服务器类型、添加与管理命令、作用域和安全边界，并用实际流程说明如何让 Claude 访问外部数据与工具
- **[23 · Claude Code 子代理教程：创建、选择与并行协作](./23-subagents.md)** -- 子代理的隔离上下文、配置文件、工具权限、自动选择和适用任务，帮助你把专项工作拆出去而不污染主会话
- **[24 · Claude Code 插件指南：安装、组合与分发能力](./24-plugins.md)** -- Claude Code 插件如何组合命令、Agent、Skill、Hook 和 MCP，覆盖插件安装、启用、目录结构与团队分发的基本流程
- **[25 · Claude Code 记忆系统：跨会话保留项目经验](./25-memory.md)** -- Claude Code 自动记忆与 CLAUDE.md 的区别、记忆内容的查看和管理方式，以及什么信息适合长期保留、什么不应该写入
- **[26 · Claude Code Agent Skills 教程：结构、触发与边界](./26-agent-skills.md)** -- Agent Skills 的目录结构、SKILL.md 写法、自动触发与手动调用方式，解释它和命令、子代理、Hook、MCP 的职责差异
- **[27 · Claude Code Skills 实战：安装、调用与验证](./27-skills-in-practice.md)** -- 从发现一个 Skill、安装到正确目录、触发执行到检查结果的完整流程，并说明常见路径错误、触发失败和安全检查方法
- **[28 · Claude Code skill-creator 教程：创建自定义 Skill](./28-skill-creator.md)** -- 使用 skill-creator 从需求定义、指令编写、资源组织到测试迭代创建一个自定义 Skill，帮助你把重复工作沉淀为稳定能力
- **[29 · Claude Code Agent Teams 教程：多会话团队协作](./29-agent-teams.md)** -- Agent Teams 的负责人、队友、共享任务和消息机制，覆盖启用方式、任务拆分、成本风险和适合并行协作的真实场景
- **[30 · CLAUDE.md、Skill、Hook、MCP 与 Subagent 怎么选](./30-choosing-features.md)** -- 对比 CLAUDE.md、Skill、Hook、MCP、Subagent 的触发方式、职责、上下文成本和使用场景，提供一张可直接套用的功能选择表

## 五 · 系统配置与优化

集中理解 settings.json、输出样式、Hooks、CLI、模式与检查点。

- **[31 · Claude Code settings.json 配置详解](./31-settings-json.md)** -- 用户级、项目级和本地 settings.json 的路径、优先级与常用字段，说明权限、环境变量、Hook 和插件配置如何安全落盘
- **[32 · Claude Code Output Styles 输出样式指南](./32-output-styles.md)** -- 输出样式的作用、内置样式、自定义文件结构、切换方式和适用场景，解释它如何改变回答形式而不替换系统核心能力
- **[33 · Claude Code Hooks 教程：事件、配置与自动化](./33-hooks.md)** -- Hook 的生命周期事件、匹配器、命令配置、输入输出和错误处理，帮助你在固定节点自动执行格式化、检查与安全卡点
- **[34 · Claude Code CLI 命令与参数完整参考](./34-cli-reference.md)** -- Claude Code 命令行入口、常用参数、会话恢复、模型选择、权限设置和非交互用法，适合需要快速查询准确命令的读者
- **[35 · Claude Code 控制模式：权限、模型与会话选项](./35-modes-and-control.md)** -- 启动和会话中的权限模式、模型、思考强度、自动模式与控制选项，帮助你根据任务风险和复杂度调好执行方式
- **[36 · Claude Code 斜杠命令完整指南](./36-slash-commands.md)** -- 内置斜杠命令的分类、常用参数和使用时机，覆盖会话管理、上下文、配置、诊断与扩展命令的快速调用方式
- **[37 · Claude Code Checkpoints 检查点与回滚指南](./37-checkpoints.md)** -- 检查点如何保存文件修改、查看历史和恢复到安全状态，说明它能回滚什么、不能替代什么，以及和 Git 的正确配合方式

## 六 · 高级参考与实战

把插件、浏览器、并行、Git、CI、SDK 和综合项目串成工程流程。

- **[38 · Claude Code 插件参考：目录、清单与发布](./38-plugins-reference.md)** -- 插件目录结构、manifest、组件路径、版本与 marketplace 分发方式，适合需要制作、检查或发布 Claude Code 插件的读者
- **[39 · Claude Code 实战入门：从需求到交付完整流程](./39-getting-started-practice.md)** -- 用一个真实需求串起项目阅读、计划确认、代码修改、测试验证和结果汇报，展示 Claude Code 任务从开工到交付的标准流程
- **[40 · Claude Code 操作 Chrome 浏览器指南](./40-chrome.md)** -- Claude Code 连接并操作 Chrome 的安装条件、页面交互、权限确认、调试场景和安全边界，说明浏览器能力适合解决哪些任务
- **[41 · Claude Code 并行任务：子代理、Worktree 与团队](./41-parallel-tasks.md)** -- 多会话、子代理、Agent Teams 和 Git Worktree 的并行方式、隔离程度与成本差异，帮助你选择不会互相覆盖的协作方案
- **[42 · Claude Code 环境变量完整参考](./42-env-vars.md)** -- Claude Code 常用环境变量的作用、配置位置、优先级与排错方式，覆盖认证、模型、网络、日志和运行行为等关键开关
- **[43 · Claude Code Git 工作流：提交、分支与代码审查](./43-git-workflow.md)** -- 让 Claude Code 安全参与查看差异、创建分支、准备提交、处理冲突和审查变更的流程，强调每一步的人类验收边界
- **[44 · Claude Code GitHub Actions 集成教程](./44-github-actions.md)** -- 在 GitHub Actions 和 PR 中调用 Claude Code 的安装、认证、工作流配置、触发方式与安全注意事项，适合仓库自动化场景
- **[45 · Claude Agent SDK 教程：把代理能力接入程序](./45-agent-sdk.md)** -- Claude Agent SDK 的核心概念、安装、调用、工具、会话与权限控制，帮助开发者把 Claude Code 的代理循环嵌入自己的应用
- **[46 · Claude Code 开发环境配置指南](./46-dev-config.md)** -- 终端、Shell、开发容器、网络代理和项目初始化等环境配置，帮助你减少命令执行差异并让 Claude 在稳定环境里工作
- **[47 · Claude Code Voice 语音听写使用指南](./47-voice.md)** -- Voice 语音听写的启用方式、快捷键、语言识别、编辑流程和适用场景，说明它与真正的语音对话模式有什么区别
- **[48 · Claude Code 综合实战：从零开发到上线](./48-capstone-project.md)** -- 从需求拆解、项目初始化、功能开发、测试、Git 提交到上线检查的完整案例，把前面学到的 Claude Code 能力串成可复用流程

## 七 · 收尾与查阅

用最佳实践、反模式、故障排查和术语表巩固整套方法。

- **[49 · Claude Code 最佳实践：稳定交付的工作方法](./49-best-practices.md)** -- 任务拆分、上下文控制、先计划后修改、验证闭环、权限边界和规则维护等最佳实践，帮助你减少返工并提升代理结果质量
- **[50 · Claude Code 反模式：常见错误用法与修正](./50-anti-patterns.md)** -- 模糊指令、无限上下文、过度授权、跳过验证和错误配置等常见反模式，解释问题为什么发生以及应该如何修正
- **[51 · Claude Code 常见问题排查完整指南](./51-troubleshooting.md)** -- 安装、登录、网络、权限、MCP、IDE、模型和性能等常见故障的定位顺序与解决方法，适合遇到问题时按现象快速查找
- **[52 · Claude Code 术语表：小白也能看懂的概念解释](./52-glossary.md)** -- 解释 Agent、Context、Token、MCP、Hook、Skill、Subagent、Checkpoint 等高频术语，并给出它们在 Claude Code 中的实际作用
- **[53 · Claude Code 制作 Remotion 视频实战](./53-remotion-video.md)** -- 使用 Claude Code 和 Remotion 从脚本、组件、动画、素材到渲染输出制作程序化视频，说明适用场景、开发流程与验收方法

## 关于这套教程

作者：stormzhang。教程持续对照官方文档维护，页面中的「最近验证」表示内容最后一次完成官方资料核对或真实运行检查的日期。

项目源码与问题反馈：https://github.com/stormzhang/ai-coding-guide
