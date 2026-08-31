---
seoTitle: "用 npx 启动 DeepSeek Harness：固定版本运行 Web UI"
description: "使用固定 npm 版本从项目目录启动 DeepSeek Harness Web UI，验证版本、端口、首页、停止方式和回环网络边界，并掌握端口冲突、缓存、SSH 与启动失败的排查方法。"
lastVerified: "2026-08-31"
author: stormzhang
dshVersion: "0.1.1-rc.2"
verificationLevel: "V1"
verificationStatus: "runtime-passed-owner-experience-confirmed"
stability: "version-bound"
officialSources:
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/README.zh.md"
  - "https://deepseek-harness.github.io/deepseek-harness/guide/quickstart"
  - "https://github.com/deepseek-ai/deepseek-harness/blob/b150a551b8d465e31e418e1b2eaf5e79bbb7d28e/packages/bundle/web-app/README.zh.md"
related:
  - "./06-requirements.md"
  - "./08-web-ui-workspace.md"
  - "./09-deepseek-provider.md"
evidence:
  - "references/deepseek-harness/ARTICLE-SOURCE-MAP.md：第 07 篇"
  - "references/deepseek-harness/RUNTIME-BASELINE.md"
  - "2026-08-27 北京时间隔离目录重跑 CLI、Web 与 HTTP 边界"
---

# 07 · 用 `npx` 启动 Harness

> 📚 **系列导航**：上一篇 [06 · 环境准备](./06-requirements.md) 已经检查 Node.js、npm 和工作区。这一篇只做一件事：把固定版本的 Web UI 稳稳跑起来。

> [!WARNING] Developer Preview
> 本文命令、CLI 帮助、Web 首页与网络接口已在 `0.1.1-rc.2` 上验证；npm `latest` 安装口径截至 2026-08-31 与本教程固定版本一致。

官方启动命令很短：

```bash
npx @deepseek-ai/dsh web
```

但教程不会直接让你复制这一行就结束。

**Developer Preview 阶段，最重要的不是少打十几个字符，而是把版本、目录、地址和停止方式全部钉住。** 否则同一篇教程隔几天再跑，拿到的可能已经不是同一套程序。

**看完这一篇，你会拿到：**

- 一条固定版本、固定回环地址、不会自动开浏览器的启动命令
- 知道首次 `npx` 到底下载了什么，以及为什么不用全局安装
- 会根据启动输出找到真实访问地址
- 用 HTTP 状态和页面标题确认 Harness 确实启动成功
- 会安全停止服务，并处理端口冲突、缓存和 SSH 场景
- 理解 `127.0.0.1`、`--trusted-host` 与远程暴露之间的安全边界

---

## 01 先确认你将启动哪个版本

在工作区终端执行：

```bash
npm view @deepseek-ai/dsh@0.1.1-rc.2 version
```

预期输出：

```text
0.1.1-rc.2
```

再检查 CLI 本身：

```bash
npx --yes @deepseek-ai/dsh@0.1.1-rc.2 --version
```

预期输出仍然是：

```text
0.1.1-rc.2
```

第一条查询 npm 元数据，不安装程序；第二条会让 `npx` 获取固定包并运行它。`--yes` 只是不再询问是否下载临时包，不会把 Harness 全局安装到系统。

| 写法 | 得到什么 | 教程是否使用 |
|---|---|---|
| `npx @deepseek-ai/dsh web` | npm 当时解析出的默认版本 | 官方最短命令，长期复现不够稳定 |
| `npx @deepseek-ai/dsh@latest web` | 当时的 `latest` | 明确但仍会随时间变化 |
| `npx --yes @deepseek-ai/dsh@0.1.1-rc.2 web` | 本教程固定版本 | 本轮推荐 |
| 全局安装后运行 `dsh` | 取决于全局安装状态 | 本批次不推荐，容易忘记真实版本 |

2026-08-27 查询时，`latest` 和 `next` 都是 `0.1.1-rc.2`。**固定版本不是因为 latest 今天有错，而是为了保证明天仍能复现今天的行为。**

> 💡 **一句话总结**：先查 npm 版本，再让 CLI 自报版本；两次都得到 `0.1.1-rc.2`，才能继续。

---

## 02 在正确目录启动 Web UI

先进入上一篇创建的练习目录。

macOS／Linux：

```bash
cd ~/dsh-playground
```

Windows PowerShell：

```powershell
Set-Location "$HOME\dsh-playground"
```

然后启动：

```bash
npx --yes @deepseek-ai/dsh@0.1.1-rc.2 web \
  --no-open \
  --host 127.0.0.1 \
  --port 3080
```

Windows PowerShell 使用反引号续行：

```powershell
npx --yes @deepseek-ai/dsh@0.1.1-rc.2 web `
  --no-open `
  --host 127.0.0.1 `
  --port 3080
```

预期看到：

```text
dsh web: http://127.0.0.1:3080
```

现在这个终端会持续占用，说明 Harness Host 正在前台运行。不要在同一个终端继续输入其他命令；需要检查页面时，另开一个终端窗口。

这几个参数分别解决什么问题：

| 参数 | 作用 | 为什么首篇显式写 |
|---|---|---|
| `web` | 启动 Web Profile | 明确使用浏览器界面 |
| `--no-open` | 不自动打开默认浏览器 | 让启动和浏览器走查分开，输出更可控 |
| `--host 127.0.0.1` | 只绑定本机回环地址 | 避免意外暴露到局域网 |
| `--port 3080` | 使用固定端口 | 访问地址容易记，便于排错 |

2026 年 8 月 27 日，本教程第一次启动固定版 Harness，使用的是 macOS ARM64 和 `npx @deepseek-ai/dsh@0.1.1-rc.2`。CLI 正确返回 `0.1.1-rc.2`，Web 自动选择空闲端口并打印 `dsh web: http://127.0.0.1:<port>`，首页随后返回 HTTP 200。首次下载耗时当时没有单独留日志，所以这里不编数字；从版本检查到网页响应这一条启动链路是一次跑通的。

> 💡 **一句话总结**：从目标工作区启动，并显式固定版本、回环地址和端口，后续所有现象才有稳定坐标。

---

## 03 动手：验证页面真的活着

先在浏览器打开：

```text
http://127.0.0.1:3080
```

如果暂时不看视觉，也可以在第二个终端做 HTTP 检查。

macOS／Linux：

```bash
curl --silent --show-error --head http://127.0.0.1:3080/
```

预期状态和类型：

```text
HTTP/1.1 200 OK
content-type: text/html; charset=utf-8
```

Windows PowerShell：

```powershell
$response = Invoke-WebRequest http://127.0.0.1:3080/
$response.StatusCode
$response.Content -match '<title>DeepSeek Harness</title>'
```

预期输出：

```text
200
True
```

本次复验还确认首页 HTML 标题是 `DeepSeek Harness`。HTTP 200 只能证明服务和静态页面可用，不能证明按钮、工作区选择和模型请求都正常；后面会分别验收。

**类比：HTTP 200 相当于店门开着，不等于厨房、收银和配送全都正常。** 一层一层验证，比打开页面后凭感觉判断靠谱得多。

> 💡 **一句话总结**：地址能返回 HTTP 200，页面标题又正确，才算 Web Host 的第一层启动成功。

---

## 04 端口冲突时，用 `--port 0` 让系统选择

如果 `3080` 已被其他进程占用，启动会失败。最快的验证方式不是随手猜另一个端口，而是让操作系统选择空闲端口：

```bash
npx --yes @deepseek-ai/dsh@0.1.1-rc.2 web \
  --no-open \
  --host 127.0.0.1 \
  --port 0
```

本次实测得到：

```text
dsh web: http://127.0.0.1:<系统选择的端口>
```

`<系统选择的端口>` 每次可能不同，要以终端打印值为准。这个方法很适合判断「程序不能启动」到底是 Harness 问题，还是固定端口冲突。

| 现象 | 优先判断 | 处理方式 |
|---|---|---|
| `3080` 启动失败，`--port 0` 成功 | 端口冲突 | 关闭占用进程或长期换端口 |
| 两种端口都失败 | 不是单纯端口问题 | 查看完整错误、Node.js 与包下载状态 |
| 终端打印 URL，浏览器拒绝连接 | 地址或运行环境不一致 | 确认进程仍在、浏览器访问打印出的地址 |
| 浏览器打开旧页面 | 可能访问了另一个旧进程 | 停掉旧进程，核对端口与 CLI 版本 |

> 💡 **一句话总结**：`--port 0` 是排除端口冲突的诊断工具，实际访问必须使用启动日志打印的端口。

---

## 05 停止、重启与 `npx` 缓存

回到运行 Harness 的终端，按：

```text
Ctrl + C
```

进程结束后，再访问原地址应该连接失败。**这才说明你停掉的是正确进程。**

重新运行同一条固定版本命令时，`npx` 通常会复用本地缓存，因此第二次启动往往更快。缓存只影响下载速度，不应该改变显式指定的包版本。

如果怀疑拿到的不是预期版本，永远回到这条命令：

```bash
npx --yes @deepseek-ai/dsh@0.1.1-rc.2 --version
```

不要先删整个 npm 缓存。清缓存影响其他项目，而且多数问题与缓存无关。先核对版本、完整报错和网络，再决定是否需要更窄的缓存排查。

本教程这次验收真正遇到的不是端口占用，而是默认 npm cache 里存在 root-owned 文件，执行 `npm view` 时直接报 `EPERM`。我决定不照错误提示改整个用户缓存的所有权，也不清空缓存，而是让验收切到 `/tmp` 下的独立 cache，同一组版本和 dist-tag 查询马上通过。端口则始终用 `--port 0` 让 Harness 自动选择，避免把缓存问题和旧进程、固定端口冲突混在一起排查。

> 💡 **一句话总结**：用 `Ctrl + C` 明确停止前台 Host；重启后先查版本，不要把清空全局缓存当作第一反应。

---

## 06 本机、SSH 与暴露边界

默认 `127.0.0.1` 只允许本机访问，这是当前最合适的起点。

固定版 Web CLI 会拒绝 `--host 0.0.0.0`。这不是少了一个方便参数，而是产品当前主动收紧边界：配置、凭据、工作区和 Agent 操作都属于高权限界面，不能把它当成普通静态网页直接暴露到公网。

官方 README 还区分了本机与 SSH 启动：

- 本机启动默认会尝试打开默认浏览器。
- SSH 环境只打印宿主机 URL，不自动打开浏览器。
- 本地转发地址由 SSH 客户端或编辑器负责。
- `--trusted-host` 是浏览器信任栅栏的额外 authority，不等于登录认证，也不应该被理解为「开放公网」。

如果通过 SSH 使用，安全思路应该是：Host 仍绑定远端 `127.0.0.1`，再通过 SSH 端口转发到本机。不要为了省一步，把高权限 Web UI 暴露在任意可达网卡上。

本次网络边界复验得到：

| 请求 | 结果 | 含义 |
|---|---|---|
| `GET /` | HTTP 200 | 首页可用 |
| `GET /api` | HTTP 404 | 没有无边界 API 根路由 |
| 普通 `GET /api/events.mux` | HTTP 426 | 该路径要求升级为 WebSocket |
| 无 JSON 类型的 `POST /api/host.describe` | HTTP 415 | RPC 要求 `application/json` |

这些结果是协议诊断，不是普通用户的操作步骤。它们说明 Web 页面背后还有 Host 信任与传输边界，绝不是一个随便上传静态文件就能运行的前端。

这轮验收我明确选择本机回环地址，没有把 Web UI 直接暴露到局域网或公网。固定版在测试中对 `--host 0.0.0.0` 直接以退出码 1 拒绝，而 `127.0.0.1` 下首页、RPC 和 WebSocket 都能正常工作。我的判断很简单：Harness 能读文件、跑命令、申请权限，远程访问不能只图方便；真要在 SSH 环境使用，我会优先做端口转发，而不是先放开监听地址。

> 💡 **一句话总结**：`127.0.0.1` 是默认安全边界；SSH 用转发，不要把 `--trusted-host` 误当成身份认证或公网部署方案。

---

## 07 启动失败排错表

| 现象 | 先执行什么 | 判断 |
|---|---|---|
| npm 下载失败 | `npm view @deepseek-ai/dsh@0.1.1-rc.2 version` | npm Registry 或网络问题 |
| CLI 版本不对 | 固定版 `--version` | 命令、别名或全局旧版本混用 |
| `web` 参数报错 | 固定版 `web --help` | 参数位置或当前版本不支持 |
| 端口占用 | 改为 `--port 0` | 固定端口冲突 |
| 页面拒绝连接 | 看启动终端是否仍运行 | Host 已退出或访问地址错误 |
| 页面打开但输入框不可用 | 进入下一篇选择工作区 | 这是全新 UI 的预期状态 |
| 页面正常但模型请求失败 | 进入第 09 篇配置 Provider | 启动成功不等于凭据已配置 |

查看固定版 Web 参数：

```bash
npx --yes @deepseek-ai/dsh@0.1.1-rc.2 web --help
```

当前帮助中应包含：

```text
--host <host>
--no-open
--port <port>
--trusted-host <authority...>
```

> 💡 **一句话总结**：下载、CLI、端口、页面、工作区和 Provider 是六层不同故障，按层排查比反复重装有效。

---

## 小结

这一篇完成了 Web Host 的最小启动闭环：

1. npm 元数据与 CLI 都确认版本为 `0.1.1-rc.2`。
2. 从目标工作区运行固定版本，不依赖全局安装。
3. `--no-open --host 127.0.0.1 --port 3080` 把行为写清楚。
4. HTTP 200 和页面标题证明首页已启动。
5. 端口冲突可以用 `--port 0` 快速排除。
6. `Ctrl + C` 停止前台服务，重启时继续核对版本。
7. Web UI 保持回环访问，远程使用走安全转发思路。

你现在应该已经能独立启动和停止 Harness，并判断问题卡在下载、进程、端口还是页面层。

---

下一篇：[08 · Web UI 界面与工作区](./08-web-ui-workspace.md)。页面打开只是开始，接下来要让 Harness 知道自己究竟在哪个项目里工作。
