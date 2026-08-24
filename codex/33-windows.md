---
seoTitle: "Codex Windows 使用指南：原生环境与 WSL"
description: "Codex 在 Windows 原生沙箱、PowerShell 和 WSL 中的安装、路径、权限与常见错误，帮助你选择更稳定的运行方式，并给出适合中文开发者直接照做的操作思路、检查方法与风险边界。"
published: "2026-06-12"
lastVerified: "2026-06-20"
author: stormzhang
officialSources:
  - https://developers.openai.com/codex/windows
related:
  - ./30-models.md
  - ./32-migrate-from-claude-code.md
---
# 33 · Windows 使用要点：原生还是 WSL，到底怎么跑才省心

> 📚 **系列导航**：上一篇〔[32 从 Claude Code 迁移](32-migrate-from-claude-code.md) 〕讲的是「用熟了 Claude Code，怎么平滑切到 Codex」。这一篇专门给 Windows 用户开小灶——前面三十多篇默认你在 Mac 或 Linux 上敲命令，可咱们国内一大半人手里是 Windows，路径、换行、沙箱全是另一套规矩。下一篇〔[34 综合实战](34-capstone.md) 〕把整个 Codex 篇学到的东西串成一个完整项目跑一遍。

说个我自己干过的蠢事。

2026 年 3 月我第一次在公司那台 Windows 11 笔记本上装 Codex，图省事没装 WSL，直接原生 PowerShell 里就跑了。装是装上了，结果一让它改文件就报错——我盯着日志看了半天，才发现是我从 Mac 那边 `git clone` 下来的项目，里面一堆脚本是 `LF` 换行，Git 在 Windows 上默认把它们全转成了 `CRLF`，Codex 改完一保存，整个文件的 diff 全是 `^M`，等于每一行都被「改」过一遍。我还以为是 Codex 出 bug 了，提了半天 issue 草稿，最后发现是自己根本没搞懂 Windows 那套换行规矩。

**说句实话，Codex 在 Windows 上现在真能原生跑，体验也不差，但它跟 Mac/Linux 不是一回事。** 沙箱机制是 Windows 专属的两套，路径是反斜杠，换行符是历史包袱，这些坑你不提前知道，迟早会踩。

**先划清范围：本篇讲的是 Windows 下的 Codex CLI，也就是终端里的用法。** 如果你只用 Codex 桌面 App 或 IDE 扩展，装好就能用，基本没这些事——下面的 PowerShell、WSL、沙箱模式、路径换行排查，全是 CLI 场景的门道。

这一篇就把 Windows 上的门道讲清楚：**怎么装、原生还是 WSL 怎么选、Windows 特有的三个坑怎么躲、沙箱跟 Mac/Linux 差在哪，最后亲手在 PowerShell 里跑通一个最小例子。**

**看完这一篇，你会拿到：**

- 一句话结论：Windows 上最省心的跑法到底是哪种，先别纠结
- 用官方 PowerShell 脚本把 Codex 装上的完整步骤，外加几个 Windows 必备前置依赖
- 「原生 PowerShell vs WSL2」的取舍清单：你是哪种人、该选哪条路
- Windows 特有的三个坑——路径反斜杠、`CRLF` 换行、`Everyone` 权限警告——怎么提前躲掉
- Windows 沙箱的 `elevated` / `unelevated` 两种模式跟 Mac/Linux 的差异，以及报错 `1385` / `1223` 怎么办
- 一套能照做的动手流程，在 Windows 上从零跑通第一个 Codex 任务

> ⚠️ 命令、配置项、默认行为一律以 Codex [官方文档](https://developers.openai.com/codex/windows) 为准；模型名、套餐范围、版本号会随更新变动，以你本地 `codex --help` 和实际界面为准。本篇所有命令我都标了在 PowerShell 还是 WSL shell 里跑，别跑错地方。

---

## 01 一句话结论：Windows 上最省心的跑法

先把结论甩出来，省得你纠结半天：

**默认就用「原生 Windows + `elevated` 沙箱」，这是官方推荐的跑法，速度最快、安全性不打折。** 只有当你的工作流本来就泡在 Linux 里、或者两种原生沙箱模式在你公司电脑上都跑不起来时，才退回去用 WSL2。

我知道网上很多老教程（包括一些中文站）会告诉你「Windows 上推荐 `unelevated`」「推荐先装 WSL」——**这些说法过时了。** 官方文档现在白纸黑字写着：原生 Windows 沙箱性能最好、速度最快，同时安全性跟其他平台一样；`elevated` 是首选，`unelevated` 只是退路。

**类比：装宽带。** 师傅上门先给你接光纤入户（`elevated`），这是最快最稳的方案；只有当你家楼里没光纤、布线被物业卡住了，才退而求其次拉根网线走 LAN（`unelevated`）；实在都不行，才考虑搬去隔壁有网的房间办公（WSL2）。多数人家里直接接光纤就完事，根本用不着折腾后两种。

实际场景里，这个选择对应三类人：

- **你就想在 Windows 上正常写代码、用 Windows 的工具链**——原生跑，`elevated`，不用想。
- **你是公司发的「管控电脑」，IT 锁死了管理员权限**——原生 `elevated` 装不上，先用 `unelevated` 顶着，同时找 IT 报备。
- **你本来项目就在 WSL 里、用惯了 Linux 工具**——直接进 WSL2 跑，别在 Windows 和 Linux 之间反复横跳。

> 💡 一句话总结：默认「原生 Windows + `elevated`」，WSL2 是给「本来就活在 Linux 里」的人留的，不是默认起手式。

---

## 02 安装与前置依赖

Windows 上装 Codex CLI，官方最直接的路子是用自带的 PowerShell 安装脚本，一行就能装上。

**在 PowerShell 或 Windows Terminal 里跑：**

```powershell
powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"
```

如果你电脑上已经装了 Node.js，也可以走 npm 这条官方备选路径：

```powershell
npm install -g @openai/codex
```

装完关掉终端重开一个，让 `PATH` 生效，然后验证：

```powershell
codex --version
```

预期输出是一行版本号（具体数字以你装的为准），格式大致是 `codex 0.x.x`，能打印出来就说明装好了。

> 想要图形界面而不是命令行？官方提供 Windows 桌面版 Codex app，直接从 Microsoft Store 下载；不想打开 Store 界面也可以用 `winget install Codex -s msstore` 装。本篇讲的是 CLI，下面的命令都围绕 `codex` 命令行展开。

**几个前置依赖，提前备齐能少踩坑：**

| 依赖 | 为什么需要 | 怎么搞定 |
| --- | --- | --- |
| Windows 11（推荐）| 官方首选基线，最稳 | 升级到 Win 11；Win 10 也能跑但要 1809 或更新版本 |
| `winget` 可用 | 装 C++ 构建工具、桌面版 app 都会用到它 | 缺了就更新 Windows 或装 App Installer |
| 管理员审批权限 | `elevated` 沙箱的初始化要它 | 自己的电脑直接点「同意」；公司电脑可能被锁 |
| C++ 构建工具（用 IDE 扩展时）| 部分原生依赖要编译 | `winget install --id Microsoft.VisualStudio.2022.BuildTools -e` |

我自己第一次在 Windows 10 上装的时候，IDE 扩展装好了却一直「转圈不响应」，折腾了快二十分钟才在官方文档的排障节里翻到——是缺了 Visual Studio Build Tools 的 C++ 工作负载。补上那条 `winget install` 命令、重启 VS Code，立马就好了。**所以如果你也用 VS Code 里的 Codex 扩展，建议一开始就把 C++ 构建工具备上。**

> 💡 一句话总结：一条 PowerShell 安装脚本搞定 Codex CLI 主体，前置就盯紧 Windows 版本、`winget` 可用、管理员权限三样。

---

## 03 WSL vs 原生 PowerShell：你是哪种人

这是 Windows 用户最该先想明白的一道选择题。别看网上吵得凶，标准其实很简单：**看你的代码和日常工作流活在哪儿。**

**类比：在哪个城市落户。** 原生 PowerShell 就是「在 Windows 这座城落户」，办事走 Windows 那套流程，最快最顺；WSL2 是「在城里租了套 Linux 的房子」，你想用 Linux 的工具、跑 Linux 的脚本，就钻进去办公。两边都能住，但你总不能每天从 Windows 这间办公室跑去 Linux 那间再跑回来——选定一个主工位更省心。

直接上对照表：

| 维度 | 原生 PowerShell | WSL2 |
| --- | --- | --- |
| 安装难度 | ✅ 一条安装脚本搞定 | ❌ 还要先 `wsl --install` 装发行版 |
| 速度 | ✅ 原生最快 | 文件 I/O 稍慢，跨盘符更慢 |
| 沙箱实现 | Windows 专属（`elevated`/`unelevated`）| 走 Linux 的 `bubblewrap` 沙箱 |
| 适合的工具链 | ✅ Windows 原生工具、`.exe` | ✅ Linux 原生工具、bash 脚本 |
| 适合谁 | 大多数 Windows 用户 | 项目本来就在 Linux 里的人 |

**什么时候选原生 PowerShell？** 你就是个普通 Windows 用户，平时用 VS Code 写代码、用 Windows 的 Git——直接原生，别装 WSL 给自己加负担。

**什么时候选 WSL2？** 三种情况：你需要 Linux 原生工具链；你的仓库和工作流本来就在 WSL2 里；或者两种原生沙箱模式在你电脑上都跑不通。

要走 WSL2，先在「管理员身份的 PowerShell」里装：

```powershell
wsl --install
wsl
```

进了 WSL shell 之后，**Codex 要在 Linux 里重新装一遍**（Windows 上装的那个不算）：

```bash
curl -fsSL https://chatgpt.com/codex/install.sh | sh
codex
```

这里有个我 2026 年 4 月踩过的坑：我图方便把仓库放在了 `/mnt/c/Users/...` 下（也就是 Windows 的 C 盘，从 WSL 里挂载访问），结果 Codex 跑起来卡得要命，一个 `git status` 都要等好几秒。后来才知道，**WSL 访问 Windows 挂载盘（`/mnt/c/...`）的文件 I/O 特别慢**，得把仓库挪到 Linux 原生的主目录下：

```bash
mkdir -p ~/code && cd ~/code
git clone https://github.com/your/repo.git
cd repo
```

挪完之后速度立马正常。需要从 Windows 这边访问这些文件时，在资源管理器里输 `\\wsl$\Ubuntu\home\<user>` 就能进去（把 `<user>` 替换成你的 Linux 用户名）。另外提一句，WSL1 从 Codex `0.115` 起就不支持了（沙箱换成了 `bubblewrap`），你装 WSL 的时候默认就是 WSL2，不用特意操心。

> 💡 一句话总结：代码在 Windows 就用原生 PowerShell，代码在 Linux 就用 WSL2 并把仓库放进 `~/` 而不是 `/mnt/c`。

---

## 04 Windows 特有的三个坑：路径、换行、权限

这一节是本篇的核心。Mac/Linux 用户碰不到的坑，Windows 上一个不少。

### 坑一：路径反斜杠

Windows 路径用反斜杠 `C:\Users\You\project`，Unix 用正斜杠 `/home/you/project`。Codex 大多数时候会自动处理这个差异，但有两个地方你得自己上心：

- **写进 `~/.codex/config.toml` 的路径**，建议老老实实写绝对路径，比如 `C:\absolute\directory\path`。
- **给 Codex 加沙箱可读目录时**，路径必须是「已存在的绝对目录」。命令是这个（**在 Codex 交互界面里输**）：

```text
/sandbox-add-read-dir C:\absolute\directory\path
```

执行成功后，本次会话里后续在沙箱中跑的命令就能读这个目录了。注意这条只对当前会话有效，重开一个会话要重新加。

### 坑二：CRLF 换行符（开篇我栽的那个）

先说清楚：这其实是 Windows 上用 Git 的通用换行问题，不是 Codex 自己的毛病，但 Codex 会改文件，所以你特别容易在它身上撞见。

**类比：南北方插座。** `LF`（Unix 换行）和 `CRLF`（Windows 换行）就像两国不同制式的插座，看着都是「换行」，底层多了个 `\r`。你从 Mac/Linux 同事那 clone 来的项目是 `LF`，到了 Windows 上 Git 可能默认给你转成 `CRLF`，Codex 一改一保存，整个文件每行都「变」了，diff 直接爆炸。

我当时就是这么栽的。解决办法是统一换行策略，最省事的是在项目根目录放个 `.gitattributes` 文件，让 Git 别瞎转：

```text
* text=auto eol=lf
```

或者全局把 Git 的自动转换关掉（**在 PowerShell 里跑**）：

```powershell
git config --global core.autocrlf false
```

**到底用哪种看团队约定**，但核心就一句：别让换行符在平台间被偷偷改掉，否则你和 Codex 的每次改动都会被换行噪音淹没。

### 坑三：`Everyone` 可写权限警告

原生跑的时候，Codex 可能会警告「某些文件夹对 `Everyone` 可写」。这不是 bug——它在提醒你：**这些文件夹的 Windows 权限太宽了，沙箱保护不住它们。** 处理办法是去掉那些文件夹的 `Everyone` 写权限，然后重启 Codex 或重跑一次沙箱初始化。不确定怎么改权限就找 IT，别硬来。

| 坑 | 症状 | 怎么躲 |
| --- | --- | --- |
| 路径反斜杠 | 配置/读目录路径不生效 | 写绝对路径，`/sandbox-add-read-dir` 用已存在的绝对目录 |
| `CRLF` 换行 | diff 里每行都被改、满屏 `^M` | `.gitattributes` 统一 `eol=lf` 或关 `core.autocrlf` |
| `Everyone` 可写 | Codex 警告文件夹权限过宽 | 去掉 `Everyone` 写权限后重启 Codex |

> 💡 一句话总结：路径写绝对、换行用 `.gitattributes` 锁死、权限警告别忽视——这三个坑提前堵上，Windows 体验跟 Mac 没差。

---

## 05 沙箱与权限：Windows 跟 Mac/Linux 差在哪

Codex 的沙箱（限制它乱写文件、乱连网的安全围栏）在每个平台实现都不一样。Mac 用系统的 `sandbox-exec`，Linux 用 `bubblewrap`，**Windows 是一套完全独立的、自己的两种模式。**

原生 Windows 上，沙箱在 `agent` 模式下会拦住「working folder 之外的文件写入」，也会拦住「没经你同意的网络访问」。你能在 `~/.codex/config.toml` 里配它（**写到配置文件**）：

```toml
[windows]
sandbox = "elevated" # 或 "unelevated"
```

两种模式的区别，我给你拆明白：

| 模式 | 强度 | 怎么实现 | 什么时候用 |
| --- | --- | --- | --- |
| `elevated`（首选）| 更强 | 专用的低权限沙箱用户 + 文件系统权限边界 + 防火墙规则 + 本地策略调整 | 默认就用它，性能和安全都最好 |
| `unelevated`（退路）| 较弱 | 从你当前用户派生一个受限令牌 + ACL 文件系统边界 + 环境级离线控制 | `elevated` 装不上时顶着用 |

**注意：这跟某些老中文教程说的「推荐 unelevated」恰恰相反。** 官方明确写 `elevated` 是首选，`unelevated` 只是当 `elevated` 因为权限受限装不上时的临时退路。两种模式默认还都开了「私有桌面」做 UI 隔离，除非你为了兼容旧的 Windows Station `Winsta0\Default` 会话桌面行为，否则别去关 `windows.sandbox_private_desktop`。

**为什么 `elevated` 可能装不上？** 它要建专用沙箱用户、改防火墙规则、调本地策略，这些动作在公司管控电脑上经常被 IT 策略拦下来。最典型的报错是 `1385`——意思是 Windows 拒绝给沙箱用户它启动命令所需的登录权限。碰到 `1385`，按官方建议这么办：

1. 找 IT 确认设备策略有没有给「Codex 创建的沙箱用户」授予所需登录权限。
2. 如果只有部分机器有问题，对比一下组策略或 OU 差异。
3. 急着干活就先切 `unelevated` 顶着。
4. 把 `CODEX_HOME/.sandbox/sandbox.log` 连同 Windows 版本一起发给团队排查。

**同一条链路上还有个兄弟报错：`1223`**（这条官方文档暂未收录，来自读者在 Win11 + npm 安装环境下的实测反馈与修复验证）。症状长这样：`elevated` 模式下一让它改文件就报 `ShellExecuteExW failed to launch setup helper: 1223`，往往还伴随几条 `libpng warning: tRNS: invalid with alpha channel`，以及一个「`codex-windows-sandbox-setup.exe` → 找不到指定的模块」的弹窗。

它跟 `1385` 是同一根因家族的不同阶段：**`1385` 是沙箱用户在「登录阶段」被组策略拒，`1223` 是 setup helper 在「启动阶段」就没起来**——出现上面那组「libpng 警告 + 找不到模块弹窗」的组合时，大概率不是 UAC 也不是杀软，而是 npm 装的 sandbox-setup 二进制或它的资源文件损坏了，重装覆盖即可（在 PowerShell 里）：

```powershell
npm.cmd uninstall -g @openai/codex
npm.cmd install   -g @openai/codex
```

装完切回 `elevated` 再试。所以碰到「`elevated` 模式不工作」，别东一榔头西一棒子，按这条链走：**先看报错码——`1385` 找 IT 放登录权限，`1223` 用 npm 重装覆盖——都解不了，再切 `unelevated` 顶着干活。**

> 🔒 有一个文件夹千万别外发：`CODEX_HOME/.sandbox-secrets/` 里是密钥，排障发日志时只发 `sandbox.log`，别把这个目录带上。

如果某条命令因为「沙箱读不了某个目录」失败了，回到第 04 节那条 `/sandbox-add-read-dir` 把目录加进去就行。

> 💡 一句话总结：Windows 沙箱是独立的两套，默认 `elevated`、装不上退 `unelevated`；`elevated` 报错先看码——`1385` 是公司策略卡登录权限找 IT，`1223` 多半是 npm 装的 setup helper 损坏、重装覆盖就好。

---

## 06 动手：在 Windows 上跑通第一个 Codex 任务

理论讲够了，来真的。下面这套流程在原生 PowerShell 里跑，从建项目到让 Codex 改文件，全程照做。

**第一步，建个最小测试项目（在 PowerShell 里）：**

```powershell
mkdir codex-win-test
cd codex-win-test
git init
"console.log('hi')" | Out-File -Encoding utf8 app.js
```

> 小提醒：老版本 PowerShell（5.1）的 `Out-File -Encoding utf8` 会给文件加个 BOM 头，可能让后面的 `git diff` 多出点编码噪音。PowerShell 7+ 可以改用 `Set-Content -Encoding utf8NoBOM` 写出不带 BOM 的文件，验证换行时更干净。

**第二步，先把换行符规矩立好**，免得踩开篇那个坑。在项目根目录建 `.gitattributes`：

```text
* text=auto eol=lf
```

**第三步，启动 Codex：**

```powershell
codex
```

第一次跑原生沙箱，Windows 可能弹 UAC 管理员提示（这是 `elevated` 在做初始化）。自己的电脑点「同意」；如果是被锁的公司电脑点不了，Codex 通常会自动退到 `unelevated`，你会看到一行提示说切换了模式——这是正常的退路，能继续干活。

**第四步，给 Codex 派个最小任务**，在交互界面里输入：

```text
把 app.js 里的 'hi' 改成 'hello, codex on windows'
```

**预期输出**：Codex 会读 `app.js`、给出一段 diff、请求你批准（默认权限模式下会问你一句），你确认后它落盘。这时去看文件：

```powershell
Get-Content app.js
```

应该看到：

```text
console.log('hello, codex on windows')
```

**第五步，验证换行没被搞乱**——这是 Windows 上最该养成的习惯：

```powershell
git diff
```

如果你前面 `.gitattributes` 立对了，diff 里**只有那一行内容变化**，不会满屏 `^M` 或者「整个文件都被改」。要是 diff 爆了，回第 04 节坑二重新处理换行。

我自己每次在新 Windows 机器上配 Codex，都会先跑一遍这个最小流程，**特别是最后那步 `git diff` 验证换行**——这是我用真金白银（一个下午的 issue 草稿）换来的肌肉记忆。

> 💡 一句话总结：建项目 → 立 `.gitattributes` → 启动 → 派最小任务 → 用 `git diff` 验换行，五步跑通，Windows 上就算正式上路了。

---

## 小结

回头看这一篇，核心就这么几条：

- **默认跑法**：原生 Windows + `elevated` 沙箱，官方首选，最快且安全不打折；WSL2 留给「本来就活在 Linux 里」的人。
- **安装**：一条官方 PowerShell 脚本搞定 CLI，前置盯紧 Windows 版本、`winget` 可用、管理员权限，用 IDE 扩展再备 C++ 构建工具。
- **三个特有坑**：路径写绝对、换行用 `.gitattributes` 锁成 `LF`、`Everyone` 权限警告别忽视。
- **沙箱差异**：Windows 是独立的 `elevated` / `unelevated` 两套，跟 Mac 的 `sandbox-exec`、Linux 的 `bubblewrap` 都不一样；碰到 `1385` 多半是公司策略卡登录权限，碰到 `1223` 多半是 npm 装的 setup helper 损坏、重装覆盖就好。

你现在应该能：在一台干净的 Windows 机器上把 Codex 装好、判断自己该走原生还是 WSL、提前躲掉路径和换行的坑，并跑通第一个改文件的任务还顺手验了换行。

说到底，Windows 上用 Codex 不比 Mac 难，**就是多了「换行」和「沙箱模式」这两层 Windows 特有的认知税**，交过一次就再也不踩了。

---

下一篇〔[34 综合实战](34-capstone.md) 〕是整个 Codex 篇的收官——咱们不再单点讲功能，而是把前面三十多篇学的东西（配置、模型、权限、子代理、自动化、迁移、Windows 适配……）拧成一股绳，从零跑一个完整的真实项目。

留个小思考：如果让你现在挑一个项目，从 `git clone` 到让 Codex 帮你跑通第一个功能，你会先配哪三样、先立哪条规矩？下一篇见分晓。
