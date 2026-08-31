---
seoTitle: "DeepSeek Harness 中文教程：从 Agent 使用到 Harness 开发"
description: "面向普通开发者的 DeepSeek Harness 中文教程，从模型与 Harness 的关系、安装和首次任务开始，逐步讲清 PTC 模式、Session、审批沙箱、插件架构、SDK 与 Agent 基础设施开发。"
lastVerified: "2026-08-31"
author: stormzhang
pageClass: topic-hub
---

# DeepSeek Harness 中文教程

这套教程不做成泛泛的 DeepSeek 聊天和提示词大全，而是把 **DeepSeek Harness** 放在绝对主线：从会用 Agent，到理解 Harness，再到开发自己的 Agent 基础设施。

> [!WARNING] Developer Preview
> 当前教程固定验证版本为 `@deepseek-ai/dsh@0.1.1-rc.2`，对应官方 tag `dsh-v0.1.1-rc.2`。首发 21 篇（01～21）已发布，覆盖认知、安装、首次任务、四种模式与安全边界的完整闭环；后续批次正在陆续整理发布，标「施工中」的篇目尚未放出。

## 怎么选学习路线

- **第一次使用 Harness**：按首发顺序从 01 开始，先建立认知，再完成安装、首次任务、四种模式和安全边界。
- **已经用过 Claude Code／Codex**：先看 03 的选型对比和 83 的迁移指南（施工中），再看 16～21 理解 Harness 的模式差异。
- **准备开发 Agent 基础设施**：先看 14～15 完成源码构建与版本维护；Agent Loop、Cordis、插件、SDK 与综合实战批次正在施工中。

> 💡 推荐路线：01 → 02 → 04～13 → 16～21。准备读源码或开发 Harness 时再接 14～15；第 03 篇竞品对比基于 2026-08-31 当日四家官方资料核对，竞品信息按日过期。

## 一 · 认识 DeepSeek Harness

- **[01 · DeepSeek Harness 是什么](./01-what-is-deepseek-harness.md)** -- 产品定位、核心职责与 Developer Preview 边界
- **[02 · Agent = Model + Harness](./02-agent-model-harness.md)** -- 模型与执行系统的分工，一次任务的完整链路
- **[03 · DeepSeek Harness、Claude Code、Codex、OpenCode 怎么选](./03-model-harness-comparison.md)** -- 四家同日在线核对的分层对比与决策路径
- **[04 · 模型、Agent、Harness、IDE 的关系](./04-model-agent-harness-ide.md)** -- 一张任务链地图讲清四层概念
- **[05 · DeepSeek 模型目录、API Key、价格与调用方式](./05-models-api-pricing.md)** -- 三个 V4 模型、峰谷价格与 Key 安全边界

## 二 · 安装与第一次使用

- **[06 · 环境准备](./06-requirements.md)** -- Node.js、平台、网络与工作区自检（macOS 已实测）
- **[07 · 用 npx 启动 Harness](./07-npx-start.md)** -- 固定版本启动 Web UI 与端口、缓存排错
- **[08 · Web UI 与工作区](./08-web-ui-workspace.md)** -- Host／Workspace／Session 的真实运行位置
- **[09 · 配置 DeepSeek 官方 Provider 与默认模型](./09-deepseek-provider.md)** -- 凭据只写、目录可见与真实可用
- **[10 · 配置第三方与自定义 Provider](./10-custom-provider.md)** -- 最小配置与验证（真实第三方账号待实测）
- **[11 · 跑通第一个代码任务](./11-first-task.md)** -- 真实模型完成首个任务并独立验收
- **[12 · 文件修改、命令与审批](./12-files-shell-approval.md)** -- 工具管线与四种审批路径
- **[13 · Session 与 Trajectory](./13-session-trajectory.md)** -- 会话保存、轨迹查看与重启恢复
- **[14 · 从源码安装与本地开发](./14-source-install.md)** -- 固定 tag 冻结安装、类型检查与构建
- **[15 · 升级、降级、卸载与安装排错](./15-version-maintenance.md)** -- 版本四坐标、dist-tag 与分层排错

## 三 · 运行模式与工作流

- **[16 · 怎么给 Harness 下达任务](./16-task-instructions.md)** -- 目标、边界与验证的六段式任务合同
- **[17 · Standard Mode](./17-standard-mode.md)** -- 完整编码 Agent 模式与 25 个原生工具
- **[18 · PTC 模式（Code Mode）](./18-ptc-code-mode.md)** -- 用 `run_code` 编排工具调用
- **[19 · Minimal Mode](./19-minimal-mode.md)** -- 最小工具集与模型评测边界
- **[20 · Creator／创造模式](./20-creator-mode.md)** -- 用 Harness 扩展 Harness，32 个工具
- **[21 · 四种模式怎么选](./21-mode-selection.md)** -- 同任务四模式对照与选择树
- 22 · 探索陌生仓库（施工中）
- 23 · 定位并修复 Bug（施工中）
- 24 · 重构并保持行为不变（施工中）
- 25 · 编写测试并循环验证（施工中）
- 26 · 完成跨文件功能（施工中）
- 27 · 长任务与中途追加要求（施工中）

## 四 · Harness 核心能力（施工中）

28 · Agent Loop／29 · 文件工具／30 · Bash、PTY 与后台任务／31 · Web 搜索与访问／32 · LSP 代码导航／33 · 工具注册与执行／34 · 工具呈现方式／35 · 系统提示词组装／36 · 上下文、Token 与压缩／37 · Skills／38 · Plan Mode／39 · Goals／40 · Subagents／41 · Workflows／42 · Session 恢复与 Replay

## 五 · 安全边界（施工中）

43 · 审批策略与权限预设／44 · 沙箱、fail-closed 与命令安全

## 六 · Cordis 与 Harness 架构（施工中）

45～55：Cordis 五项原语、一切皆插件、插件树与 Profile、配置分层与 Patch、生命周期、Service 与依赖注入、Event 与 Waterfall、Session 持久化、Capability Seam、Agent Scope、错误恢复与运行时不变量

## 七 · 插件开发（施工中）

56～68：第一个插件、自定义工具、配置 Schema、工具拦截、超时重试、LLM Adapter、Agent Preset、自定义运行模式、设置卡片、Conversation Node、打包安装、测试热重载与生态发布

## 八 · SDK、实战与查阅（施工中）

69～85：Python SDK 与嵌入、Headless 自动化、定时任务、Telemetry、远程访问，以及代码审查、自动修复、内部 Agent 等完整实战，迁移指南、安全加固、最佳实践、排错速查与术语表

## 当前证据基线

- 官方资料固定到 `dsh-v0.1.1-rc.2` 与 commit `b150a551b8d465e31e418e1b2eaf5e79bbb7d28e`。
- 已完成 macOS ARM64 上的 Web／Headless、四种 Preset、Mock Agent Loop、审批回传、Trajectory 与重启恢复基线。
- 已完成 macOS ARM64 上固定 tag 的冻结依赖安装、类型污染诊断、普通／官方构建、源码 CLI 与 Web HTTP 基线。
- 已完成六个零依赖工程夹具的探索、Bug、重构、测试循环、跨文件功能与中途追加要求基线。
- 已完成核心运行时 20 个代表性文件、686 条测试，以及 Session／Fork／Replay 补充 7 个文件、208 条测试；固定 tag 未随远端 `master` 前移。
- 已完成 Cordis 与 Harness 架构基线：12 个代表性文件 280 条测试中 279 条通过；唯一未通过项为当前沙箱中的用户 Patch watcher 文件新增，已保留普通终端 HMR 后补。
- 已完成插件开发基线：同主题三阶段工作区 8 条测试、类型检查、构建与 pack dry-run 通过；固定源码 Adapter、Preset、Settings、Conversation、Timeout 与 Retry 合计 9 个文件 307 条测试通过。
- 已完成真实 DeepSeek API 三轮验收（300 余次真实模型请求）：四模式同任务对照、审批与沙箱链路、Goal／Skill／Subagent、Python SDK、红队注入零穿透、浏览器走查与普通终端沙箱回归；Windows／Linux 平台矩阵仍在后补。
