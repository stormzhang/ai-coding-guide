---
seoTitle: "DeepSeek Harness 环境准备：Node.js、平台、网络与工作区"
description: "从零检查 DeepSeek Harness 固定版本需要的 Node.js、npm、操作系统、浏览器、网络、工作区和 API Key 条件，并用跨平台命令完成安装前自检，提前排除版本与目录问题。"
lastVerified: "2026-08-31"
author: stormzhang
dshVersion: "0.1.1-rc.2"
verificationLevel: "V1"
verificationStatus: "runtime-passed-platform-matrix-pending-owner-experience-confirmed"
stability: "version-bound"
officialSources:
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/README.zh.md"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/package.json"
  - "https://deepseek-harness.github.io/deepseek-harness/guide/quickstart"
related:
  - "./05-models-api-pricing.md"
  - "./07-npx-start.md"
  - "./08-web-ui-workspace.md"
evidence:
  - "references/deepseek-harness/ARTICLE-SOURCE-MAP.md：第 06 篇"
  - "references/deepseek-harness/RUNTIME-BASELINE.md"
  - "2026-08-27 北京时间重新执行环境与 npm 元数据检查"
---

# 06 · 环境准备：Node.js、工作区与 API Key

> 📚 **系列导航**：上一篇 [05 · DeepSeek 模型目录、API Key、价格与调用方式](./05-models-api-pricing.md) 处理了在线模型与计费。这一篇先把本机环境收拾干净，下一篇再启动 Harness。

> [!WARNING] Developer Preview／平台矩阵待补
> 本文已在 macOS ARM64、Node.js `v26.7.0`、npm `11.19.0` 上复验固定版本；Windows、Linux 和 Node.js 22 尚未实测，这些平台的差异以官方文档为准。

安装 DeepSeek Harness 的命令只有一行，真正容易翻车的却是命令之前的环境。

Node.js 太旧、终端不在项目目录、端口被占用、浏览器访问错地址，最后都可能被误判成「Harness 装坏了」。更麻烦的是，`npx` 能下载并启动，不代表当前 Node.js 版本就在官方支持范围里。

**先做环境体检，再启动服务，排错成本会低一个数量级。**

**看完这一篇，你会拿到：**

- 固定版本真正声明的 Node.js 范围，以及为什么 Node.js 23 反而不在范围内
- macOS、Linux、Windows 当前能确认和仍待实测的平台边界
- 一套不会安装软件的环境检查命令
- 一个适合第一次练习的独立工作区，避免误操作真实项目
- 分清启动 Harness、打开 Web UI、调用 DeepSeek API 各自需要什么网络条件
- 一张安装前验收表，确认下一篇可以直接开跑

---

## 01 先看最低环境，不要只看「装了 Node.js」

固定快照根目录的 `package.json` 声明：

```json
{
  "engines": {
    "node": "^22.19.0 || >=24.0.0"
  },
  "packageManager": "pnpm@11.7.0"
}
```

这段范围翻译成人话是：

| Node.js 版本 | 是否落在声明范围 | 说明 |
|---|---|---|
| 20.x 及更早 | 否 | 版本过旧 |
| 22.0～22.18 | 否 | Node.js 22 至少需要 `22.19.0` |
| 22.19 及更高的 22.x | 是 | 落在 `^22.19.0` 范围 |
| 23.x | 否 | 奇数大版本没有被这条范围包含 |
| 24.x 及更高 | 是 | `>=24.0.0` |

这里有一个容易忽略的细节：2026-08-27 查询 `@deepseek-ai/dsh` 的 npm 元数据时，发布包没有返回 `engines`、`os` 或 `cpu` 字段。**因此 `npx` 没拦住你，不等于环境受官方项目声明支持。** 教程按同一 tag 的仓库根声明做前置判断，再用固定包真实运行。

`pnpm@11.7.0` 是源码仓库声明的包管理器。通过 `npx` 使用发布包时，不需要为了这一篇额外全局安装 pnpm；等后面的源码安装篇再处理它。

2026 年 8 月 28 日做源码构建验收时，我们遇到过一次很像 Node.js 或依赖版本不对的报错：`typecheck` 一次冒出 5 个 `ReactNode` 类型错误。继续排查才发现不是 Node.js 26.7.0，也不是冻结安装缺包，而是 TypeScript 向上读到了 `/tmp/node_modules` 里的 React 19 类型，与仓库锁定的 React 18 冲突。把类型解析限制回仓库的 `@types/react@18.3.31` 后检查通过。这次让我确认，版本报错不能只看最上面那行，父目录污染也要查。

> 💡 **一句话总结**：判断环境不能只看有没有 Node.js；固定版要求 Node.js `^22.19.0 || >=24.0.0`，而 `npx` 当前不会替你可靠执行这道门禁。

---

## 02 动手：检查 Node.js、npm 与当前平台

打开终端执行下面三条命令。macOS、Linux、Windows PowerShell 都可以运行：

```bash
node --version
npm --version
node -p '`${process.platform} ${process.arch}`'
```

本次 macOS ARM64 复验输出为：

```text
v26.7.0
11.19.0
darwin arm64
```

再用一条不依赖第三方包的命令检查 Node.js 是否落在固定版声明范围：

```bash
node -e 'const [major, minor] = process.versions.node.split(".").map(Number); const ok = (major === 22 && minor >= 19) || major >= 24; console.log(ok ? "Node.js version: PASS" : "Node.js version: FAIL"); process.exitCode = ok ? 0 : 1'
```

成功时预期输出：

```text
Node.js version: PASS
```

如果输出 `FAIL`，先切换到 Node.js 22.19 以上的 22.x，或者 Node.js 24 及更高版本，再继续下一篇。**不要因为命令偶尔能跑，就在不受支持的 Node.js 版本上继续堆排错变量。**

| 检查结果 | 判断 | 下一步 |
|---|---|---|
| `node: command not found` | 没安装 Node.js，或 PATH 未生效 | 安装受支持版本并重开终端 |
| Node.js 版本返回 `FAIL` | 不在固定版声明范围 | 切换版本 |
| 有 Node.js，没有 npm | 安装不完整或 PATH 混乱 | 重新安装官方 Node.js 发行版 |
| 平台与预期不同 | 可能在 Rosetta、WSL 或容器里 | 先确认实际执行环境和文件路径 |

> 💡 **一句话总结**：安装前要同时确认 Node.js 版本、npm 可用性和真实运行平台，三个条件缺一不可。

---

## 03 平台支持：代码覆盖不等于本教程已经实测

固定版的运行时实现包含三套本地执行路径：

| 平台 | 默认 Shell 路径 | 本地沙箱实现 | 本教程当前证据 |
|---|---|---|---|
| macOS | Bash | Seatbelt／`sandbox-exec` | ARM64 启动与无沙箱任务已验证；嵌套沙箱受当前环境限制 |
| Linux | Bash | 优先 Bubblewrap，否则 Landlock | 官方实现可查，尚未做真实平台矩阵 |
| Windows | PowerShell | ACL 受限令牌，官方标注为部分强制执行 | 官方实现可查，尚未做真实平台矩阵 |

这张表不等于一句「三平台全部稳定」。发布包没有用 npm 的 `os`、`cpu` 字段限制安装，源码也确实有三平台实现，但 Windows／Linux 的安装、PTY、路径、权限与沙箱仍需分别跑通。

**类比：施工图上画了三条路，不代表三条路都由咱们亲自开车验收过。** 官方源码能证明设计存在，平台实测才能证明某台机器上的完整体验。

如果你现在用的是 Windows，优先在 PowerShell 中运行，不要把 macOS／Linux 的 Bash 写法原样复制过去。如果你在 WSL 里启动，记住 Harness 进程看到的是 Linux 文件系统和路径；浏览器、端口转发与 Windows 文件路径又属于另一层，排错时要分开。

> 💡 **一句话总结**：当前版本设计覆盖 macOS、Linux、Windows，但本教程只把实际跑过的 macOS ARM64 标为已验证，其余平台不提前盖章。

---

## 04 准备一个独立工作区

Harness 的文件读取、编辑和 Shell 都围绕工作区发生。第一次练习不要直接选主目录、桌面根目录、公司大仓库或包含私密资料的目录。

推荐先建一个可丢弃的小目录。

macOS／Linux：

```bash
mkdir -p ~/dsh-playground
cd ~/dsh-playground
printf '{"name":"dsh-playground","private":true}\n' > package.json
```

Windows PowerShell：

```powershell
New-Item -ItemType Directory -Force "$HOME\dsh-playground" | Out-Null
Set-Location "$HOME\dsh-playground"
'{"name":"dsh-playground","private":true}' | Set-Content -Encoding utf8 package.json
```

检查当前位置：

macOS／Linux：

```bash
pwd
ls
```

Windows PowerShell：

```powershell
Get-Location
Get-ChildItem
```

预期结果是：当前位置指向 `dsh-playground`，目录中只有刚创建的 `package.json`。

为什么强调启动目录？官方 Web UI 指南说明，`dsh` 会把**进程启动时所在目录**作为默认文件系统位置；但全新的 Web UI 仍不会自动选中工作区，你还要在界面中添加并选择它。

工作区也不是完整安全边界。它提供 Session 的工作目录和持久分组；真正限制命令能否访问外部路径的是权限与沙箱策略。后面如果切到 `danger-full-access`，不能因为界面上选了一个小目录，就以为工作区之外绝对不可访问。

我没有让本教程直接拿正在写作的 `ai-coding-guide` 当 Harness 的试验田，而是安排每类任务使用独立的 `/tmp` 工作区。运行前先固定夹具哈希和 Git 状态，运行后再核对哪些文件变化；真实 Key 只进环境变量，独立 `DSH_HOME` 也不碰个人目录。第 11 篇任务因此能明确证明只改了 `calculator.js`，测试文件和 `package.json` 的 md5 都没变。对我来说，能重建的夹具比一句「我已经备份了」更可靠。

> 💡 **一句话总结**：第一次使用要在可丢弃的小目录中启动；工作区决定任务落点，但不能替代权限和沙箱。

---

## 05 网络、浏览器与 API Key 分成三件事

启动链路涉及三类连接，别混在一起排错：

| 连接 | 什么时候需要 | 失败时的表现 |
|---|---|---|
| npm Registry | 第一次通过 `npx` 下载固定包 | 下载超时、DNS 失败、包版本找不到 |
| 本机浏览器 → Harness Host | 打开 Web UI | `127.0.0.1` 拒绝连接、端口错误 |
| Harness Host → DeepSeek API | 发起真实模型请求 | `MISSING_CREDENTIAL`、401／403、429、网络错误 |

其中，API Key **不是启动 Web UI 的前提**。没有 Key 时，固定版仍能启动、打开页面、浏览 `deepseek-official` Provider 与默认模型目录；只有真正发起模型请求时才会失败。

API Key 也不要提前写进项目的 `.env`、Markdown 或 Shell 命令记录。第 09 篇会使用 Web UI 的只写凭据字段，它把值放进 `$DSH_HOME/.credentials.yaml`，settings 中只保存引用。

国内网络通常可以直接访问 DeepSeek 官方平台，但公司代理、DNS、TLS 检查和本机防火墙都可能改变结果。npm Registry 与 DeepSeek API 是两条独立链路：能下载包，不代表能调用模型；能打开 `127.0.0.1`，更不代表外网 API 正常。

> 💡 **一句话总结**：下载包、打开本机页面、调用模型是三段独立网络；API Key 只影响第三段，不影响 Harness 启动。

---

## 06 安装前最后检查

在准备好的工作区中执行：

```bash
node --version
npm --version
npm view @deepseek-ai/dsh@0.1.1-rc.2 version
```

固定版的预期结果是：

```text
v22.19.0 或更高的受支持版本
一个可用的 npm 版本号
0.1.1-rc.2
```

2026-08-27 的重新查询还确认：npm 的 `latest` 与 `next` 都指向 `0.1.1-rc.2`。但下一篇依然显式写版本，原因不是今天的 `latest` 错，而是它未来会变化。

开始前逐项打勾：

- [ ] Node.js 通过版本范围检查。
- [ ] `npm` 与 `npx` 可以运行。
- [ ] 固定 npm 版本查询返回 `0.1.1-rc.2`。
- [ ] 当前目录是专门准备的练习工作区。
- [ ] 知道第一次下载要访问 npm Registry。
- [ ] 知道没有 API Key 也能先启动 UI，但不能完成真实 Agent Turn。
- [ ] 没有把任何 Key 写进项目、命令或截图。

首轮真实 Key 环境使用 macOS 26.4 ARM64、Node.js v26.7.0、npm 11.19.0 和 Python 3.13.12。Harness 固定在 `0.1.1-rc.2`，并准备独立 npm cache、Python venv、`DSH_HOME` 与任务目录；最终 CLI、Web、Headless 和真实最小 Turn 都通过。整个环境准备没有单独计时，所以我不补一个虚假的「十分钟搞定」；能确认的是版本检查和隔离项全部留下了记录。

> 💡 **一句话总结**：Node.js、npm、固定包版本、独立工作区和密钥边界全部确认后，才进入真正启动步骤。

---

## 小结

这一篇解决的是启动之前的五个前提：

1. 固定版仓库声明 Node.js `^22.19.0 || >=24.0.0`。
2. `npx` 路线不要求全局安装 pnpm，源码路线以后单独讲。
3. macOS 已有真实运行证据，Windows／Linux 仍需平台补测。
4. 第一次练习应使用独立、可丢弃的工作区。
5. npm 下载、本机 Web UI 和 DeepSeek API 是三段不同连接。
6. API Key 不是启动条件，也绝不能进入代码或 Git。

你现在应该已经具备启动固定版 Harness 的环境，并且知道下一步的每条命令会在哪里产生影响。

---

下一篇：[07 · 用 npx 启动 Harness](./07-npx-start.md)。咱们固定版本、固定回环地址，真正把 Web UI 跑起来。
