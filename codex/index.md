---
seoTitle: "Codex 中文教程：从四种入口到工程化实战"
description: "面向中文初学者的 Codex 系统教程，覆盖桌面 App、CLI、IDE、Cloud、AGENTS.md、沙箱审批、MCP、Skills、自动化与综合实战。"
published: "2026-08-23"
lastVerified: "2026-08-23"
author: stormzhang
pageClass: topic-hub
---

# Codex 中文教程

这是一套面向中文初学者的 Codex 系统教程。内容覆盖桌面 App、CLI、IDE 和 Cloud 四种入口，并继续深入 AGENTS.md、沙箱审批、MCP、Skills、子代理和自动化；39 篇内容均以官方文档和真实操作为事实依据。

## 怎么选学习路线

- **第一次接触 AI 编程工具**：从第一组开始，按编号顺序学习，先跑通最小任务再碰高级配置。
- **已经会基础操作**：直接进入第三、四组，重点补齐工作流、权限、安全和扩展能力。
- **只想快速查资料**：从第七组的速查、排错和术语内容进入，再沿相关链接回到专篇。

> 💡 推荐起点：先读第 01 篇建立整体认识，再完成安装和第一个任务。遇到具体问题时，不必从头重学，按下面的分类直接跳转即可。

## 一 · 基础入门

先认识 Codex 的四种入口，完成安装、登录、计费和模型配置。

- **[01 · Codex 是什么？四种入口与使用场景](./01-what-is-codex.md)** -- Codex 桌面 App、CLI、IDE 扩展和云端 Web 四种入口的运行位置、能力差异与适用场景，帮助新手判断从哪里开始
- **[02 · Codex 核心概念：代理循环、沙箱与审批](./02-core-concepts.md)** -- 代理循环、上下文、工具、沙箱、审批和 Git 工作区等核心概念，建立理解 Codex 后续功能所需的完整基础模型
- **[03 · Codex 安装与登录：Mac、Windows、Linux](./03-install.md)** -- Codex CLI 在 Mac、Windows、Linux 和 WSL 上的安装、升级、登录与验证步骤，并整理网络和权限相关的常见问题
- **[04 · Codex 订阅与计费：套餐、额度和 API 成本](./04-pricing.md)** -- ChatGPT 套餐中的 Codex 权益、使用额度、API 计费和不同入口的成本差异，帮助你按使用强度选择合适方案
- **[05 · Codex 接入 DeepSeek 等第三方模型](./05-third-party-models.md)** -- 通过 OpenAI 兼容接口为 Codex CLI 配置第三方模型的基本方法、配置字段、验证步骤和能力差异，避免误解官方支持范围

## 二 · 各入口怎么上手

分别跑通桌面 App、CLI、IDE、Cloud，并写好 AGENTS.md。

- **[06 · Codex 第一个任务：从启动到检查修改](./06-first-task.md)** -- 进入项目、授权工作区、描述任务、审查差异和运行验证的完整流程，用最小案例帮助新手完成第一次 Codex 代理任务
- **[07 · Codex 桌面 App 完整使用指南](./07-desktop-app.md)** -- Codex 桌面 App 的项目管理、线程、Worktree、Review、自动化和本地环境能力，并说明它与 CLI、IDE 的选择差异
- **[08 · Codex CLI 入门：安装、会话与常用操作](./08-cli.md)** -- Codex CLI 的启动方式、交互界面、模型和权限选择、文件引用、差异审查与会话恢复，带你跑通终端主流程
- **[09 · Codex IDE 扩展指南：VS Code 与兼容编辑器](./09-ide.md)** -- Codex IDE 扩展的安装、登录、上下文引用、编辑模式、差异查看和常用命令，说明编辑器与 CLI 如何协同使用
- **[10 · Codex Cloud 云端任务使用指南](./10-cloud.md)** -- Codex Cloud 的仓库连接、云端环境、任务委派、结果审查和网络访问配置，帮助你判断哪些工作适合放到云端执行
- **[11 · AGENTS.md 完整指南：给 Codex 写项目规则](./11-agents-md.md)** -- AGENTS.md 的发现范围、层级覆盖、推荐内容、反模式和维护方法，帮助 Codex 在每次任务开始前读懂项目规范

## 三 · 核心交互与操作

掌握命令、提示词、常见工作流、沙箱审批与安全边界。

- **[12 · Codex 斜杠命令与快捷键完整指南](./12-slash-commands.md)** -- Codex CLI 的内置斜杠命令、键盘快捷键、Shell 模式和使用时机，覆盖会话、模型、权限、上下文与诊断操作
- **[13 · Codex 提示词写法：目标、上下文与验收](./13-prompting.md)** -- 如何向 Codex 说明目标、提供必要上下文、限定修改范围和写清验收标准，并用可复用结构减少猜测与返工
- **[14 · Codex 常见工作流：探索、修 Bug、重构与测试](./14-workflows.md)** -- 探索代码库、定位并修复 Bug、控制重构和补写测试四类高频任务的标准步骤、提示方式与结果验收方法
- **[15 · Codex 权限、沙箱与审批配置指南](./15-permissions.md)** -- Codex 文件系统和网络沙箱、审批策略、权限模式与命令规则，帮助你根据任务风险选择正确的执行边界
- **[16 · Codex 安全指南：提示词注入与权限边界](./16-security.md)** -- 提示词注入、恶意仓库、命令执行、网络访问、敏感信息和供应链风险，并给出隔离、审查与最小授权原则
- **[17 · Codex Computer Use：电脑与浏览器操作指南](./17-computer-use.md)** -- Computer Use 的工作方式、启用条件、可操作对象、网站审批和隐私风险，说明什么时候该用、什么时候不该交给代理

## 四 · 高级功能扩展

用 config.toml、记忆、MCP、子代理、Skills、插件和 Hooks 扩展能力。

- **[18 · Codex config.toml 配置详解](./18-config.md)** -- config.toml 的文件位置、优先级、模型、推理、沙箱、审批、MCP 和界面等常用字段，并提供可复制的配置思路
- **[19 · Codex Memories 与 Chronicle 记忆系统](./19-memory.md)** -- Memories 和 Chronicle 如何从会话与屏幕上下文形成长期记忆、如何查看管理，以及隐私边界和适合保留的信息类型
- **[20 · Codex MCP 教程：连接外部工具与数据](./20-mcp.md)** -- Codex 配置 MCP 服务器的文件格式、命令、认证、作用域与安全边界，并用实际流程说明如何接入外部服务
- **[21 · Codex 子代理教程：配置与并行协作](./21-subagents.md)** -- Codex 子代理的创建、角色说明、模型和工具配置、调用方式与上下文隔离，帮助你把复杂任务拆成专项工作
- **[22 · Codex Agent Skills 教程：安装、触发与创建](./22-skills.md)** -- Agent Skills 的目录结构、SKILL.md、安装位置、自动触发与手动调用，说明如何把重复流程沉淀为可复用能力
- **[23 · Codex 插件指南：安装一整套能力](./23-plugins.md)** -- Codex 插件如何打包 Skill、MCP、应用连接和工作流，覆盖安装、启用、权限、目录结构与适用场景
- **[24 · Codex Rules 与 Hooks：命令规则和生命周期自动化](./24-hooks.md)** -- Rules 如何控制沙箱外命令，Hooks 如何在生命周期节点运行确定性脚本，并说明两者的配置、边界和组合方式

## 五 · 工程化与自动化

把 Worktree、GitHub、CI/CD、codex exec 和外部集成接入团队流程。

- **[25 · Codex Worktrees 并行隔离指南](./25-worktrees.md)** -- Git Worktree 在 Codex 桌面 App 与 CLI 并行任务中的作用、创建和合并流程，以及避免文件覆盖与分支混乱的方法
- **[26 · Codex Git 与 GitHub 集成：提交和 PR 审查](./26-git-github.md)** -- Codex 参与本地 Git 差异审查、提交准备和 GitHub PR Review 的工作流，说明本地检查与远端协作记录的区别
- **[27 · Codex 自动化与 CI/CD 完整指南](./27-automation.md)** -- Codex GitHub Action、云端自动化和脚本任务的配置、认证、触发与审查流程，帮助你安全地把代理接入持续集成
- **[28 · codex exec 非交互模式：脚本与 CI 用法](./28-noninteractive.md)** -- codex exec 的输入输出、常用参数、结构化结果、恢复会话和退出码，说明如何把 Codex 稳定接入脚本和 CI
- **[29 · Codex Slack、Linear 与 SDK 集成指南](./29-integrations.md)** -- 在 Slack 和 Linear 中委派 Codex 任务，以及通过 SDK 把本地代理嵌入产品的入口、流程、认证与适用场景

## 六 · 实战与进阶

学习模型选择、提速、迁移、Windows 使用与完整项目实战。

- **[30 · Codex 模型怎么选：能力、速度与推理强度](./30-models.md)** -- Codex 可用模型、推理强度、速度和任务复杂度之间的关系，帮助你为探索、编码、审查和轻量任务选择合适档位
- **[31 · Codex 提速指南：上下文、模型与并行策略](./31-speed.md)** -- 通过减少无关上下文、改善任务拆分、选择模型、复用规则和并行隔离来提升 Codex 速度，同时保持结果可验证
- **[32 · 从 Claude Code 迁移到 Codex 完整指南](./32-migrate-from-claude-code.md)** -- 把 CLAUDE.md、Skill、MCP、Hook、权限和日常工作流迁移到 Codex 的对应方式，指出可以复用与必须重写的部分
- **[33 · Codex Windows 使用指南：原生环境与 WSL](./33-windows.md)** -- Codex 在 Windows 原生沙箱、PowerShell 和 WSL 中的安装、路径、权限与常见错误，帮助你选择更稳定的运行方式
- **[34 · Codex 综合实战：从需求到 Git 提交](./34-capstone.md)** -- 用一个 TODO 工具功能需求串起项目规则、计划、编码、测试、差异审查和 Git 提交，展示完整且可复用的交付流程

## 七 · 收尾与查阅

用速查表、最佳实践、故障排查、术语表和企业治理完成收尾。

- **[35 · Codex 命令与 config.toml 配置速查表](./35-cheatsheet.md)** -- 汇总 Codex CLI 参数、斜杠命令、权限模式、config.toml 字段和常用文件路径，适合日常使用时快速定位准确写法
- **[36 · Codex 最佳实践：稳定交付的工作方法](./36-best-practices.md)** -- 任务拆分、上下文控制、项目规则、计划确认、最小修改和验证闭环等最佳实践，帮助你减少误改与无效等待
- **[37 · Codex 常见问题排查：安装、登录与权限](./37-faq.md)** -- Codex 安装失败、无法登录、不修改文件、沙箱报错、网络和配置异常的定位顺序与解决方法，按现象快速排查
- **[38 · Codex 术语表：Agent、沙箱、Skill 与 MCP](./38-glossary.md)** -- 解释 Agent、Thread、Context、Sandbox、Approval、MCP、Skill、Subagent、Worktree 等高频术语及其实际作用
- **[39 · Codex 企业管理与治理完整指南](./39-enterprise.md)** -- 企业部署 Codex 时的管理员设置、身份、权限、托管配置、数据治理和审计要求，说明个人使用与组织治理的关键差异

## 关于这套教程

作者：stormzhang。教程持续对照官方文档维护，页面中的「最近验证」表示内容最后一次完成官方资料核对或真实运行检查的日期。

项目源码与问题反馈：https://github.com/stormzhang/ai-coding-guide
