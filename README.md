<div align="center">

# REA · 逆向工程使用手册

**Reverse Engineer Anything —— 一个命令，让你和 AI Agent 都能"看懂"任何软件**

[![npm version](https://img.shields.io/npm/v/rea-agents?style=flat-square&color=cb3837)](https://www.npmjs.com/package/rea-agents)
[![license](https://img.shields.io/badge/license-MIT-f4c430?style=flat-square)](LICENSE)

基于 [morluto/rea](https://github.com/morluto/rea) v6.5.0 · 本 README 是面向中文用户的**实操手册版**
各语言翻译版见 [docs/readme/](docs/readme/) · 原始英文版（对照参考）：[README_original.md](README_original.md)

</div>

> 阅读难度标记：🟢 新手也能做 · 🟡 需要一点命令行基础 · 🔴 进阶/开发者

---

## 目录

- [1. 这是什么，能帮你做什么](#1-这是什么能帮你做什么)
- [2. 能力清单：能分析什么，需要额外装什么](#2-能力清单能分析什么需要额外装什么)
- [3. 安装（三种方式，任选其一）](#3-安装三种方式任选其一)
- [4. 初始配置](#4-初始配置)
- [5. 基础操作：5 分钟上手](#5-基础操作5-分钟上手)
- [6. 进阶用法](#6-进阶用法)
- [7. 实战示例](#7-实战示例)
- [8. 常见问题排查（FAQ）](#8-常见问题排查faq)
- [9. 目录结构与技术栈](#9-目录结构与技术栈)
- [10. 许可与合规](#10-许可与合规)

---

## 1. 这是什么，能帮你做什么

REA（Reverse Engineer Anything）是一个**本地运行的逆向工程工具**，提供两种使用方式：

1. **命令行（CLI）**：在终端直接分析软件；
2. **MCP 服务器**：接入 Claude Code、Cursor、Codex 等 AI Agent，让 Agent 获得"拆解软件"的能力。

它回答的核心问题是：**"这个功能是怎么实现的？"** —— 不需要源码，直接对二进制、JavaScript/Electron 应用、.NET 程序集、APK、网站等目标取证，输出结论时会附带证据（Evidence）和局限说明，而不是凭空猜测。

**典型场景**

- 看到一个 App 的好功能，想搞懂它的实现思路，再写一个自己的版本；
- 分析 Electron 应用的模块结构、IPC 通道、预加载脚本边界；
- 查看 .NET 程序集的类型、方法、CIL 指令（纯静态，无需运行）；
- 排查某个软件依赖了哪些库、加载了哪些文件；
- 给 AI Agent 配一套逆向工具链，让它替你完成调查并写出实现。

**它不做什么**：不上传你的文件（分析全在本地）；不替你破解软件许可；不保证"还原出完整源码"（它给出的是证据与可验证的推断）。

> ⚠️ 合规底线：只分析**你拥有或获得授权**的软件，用于学习、互操作、安全研究等合法目的。详见 [DISCLAIMER.md](DISCLAIMER.md)。

---

## 2. 能力清单：能分析什么，需要额外装什么

| 分析目标                     | 你能得到什么                               | 额外依赖                                |
| ---------------------------- | ------------------------------------------ | --------------------------------------- |
| JavaScript / Electron 应用   | 模块、import 关系、路由、IPC、原生插件边界 | 无（有 Node.js 即可）✅ 新手友好        |
| .NET 程序集（.dll/.exe）     | 元数据、类型/方法签名、CIL 指令、依赖清单  | 无（纯静态分析）✅ 新手友好             |
| 原生二进制（exe/ELF/Mach-O） | 伪代码、汇编、字符串、符号、调用与引用     | 需已安装 **Hopper / Ghidra / IDA** 之一 |
| 网站                         | 页面结构、脚本、网络请求、截图             | Chrome 系浏览器                         |
| Android APK                  | Manifest、类、反编译方法、资源             | JADX + 完整 JDK                         |
| SQLite 数据库                | 表、列、索引、视图、触发器、样本行         | Node 24.15+ / 26+                       |
| 固件 / 流量包 / 信号记录     | 解包区域、协议字段、解码结果               | 多为 Linux 工具链（binwalk/tshark 等）  |

> 💡 **新手提示**：如果你的目标是 Electron 应用或 .NET 程序，**不需要安装任何重型引擎**，装好 Node.js 就能直接用。

---

## 3. 安装（三种方式，任选其一）

### 前置检查 🟢

```bash
node -v
```

要求 **Node.js 22.x（≥22.19）、24.x（≥24.11）或 26+**。版本不符会直接报错，先去 [nodejs.org](https://nodejs.org/) 或 nvm/fnm 升级。

### 方式 A：npx 即用（新手推荐）🟢

不安装任何东西，直接运行并完成配置向导：

```bash
npx rea-agents setup
```

向导会询问要为哪些 Agent 注册 REA、是否安装 Hopper 等，逐项确认即可。

临时执行单条命令：

```bash
npx -y rea-agents@latest analyze-javascript-application "D:/apps/example" --json
```

### 方式 B：全局安装（常用）🟢

```bash
npm install --global rea-agents
rea --help
```

装好后 `rea` 命令随处可用，后续用 `rea update` 升级。

### 方式 C：源码构建（进阶 / 离线 / 二次开发）🟡

```bash
git clone https://github.com/morluto/rea.git
cd rea
npm ci
npm run build:cached
node scripts/rea.mjs --version
```

构建完成后，用 `node scripts/rea.mjs <命令>` 代替全局 `rea` 命令。

**Windows 源码构建必看**：源码仓库不包含 Windows 原生控件文件（`native/windows/build/` 被 .gitignore 排除，且只能在 Linux 上编译）。若构建报错：

```text
REA Windows native controls are unavailable: Missing packaged Windows native file
native/windows/build/manifest.json (ENOENT)
```

按官方提示从 npm 发布包恢复这两个文件即可：

```bash
npm pack rea-agents@6.5.0
tar -xzf rea-agents-6.5.0.tgz package/native/windows/build/manifest.json package/native/windows/build/rea-windows-x64.node
mkdir -p native/windows/build
cp package/native/windows/build/manifest.json package/native/windows/build/rea-windows-x64.node native/windows/build/
npm run build:cached
```

> 💡 不想让 npm 缓存占用 C 盘？安装时指定缓存目录：`npm_config_cache=E:\npm-cache npm ci`（Windows Git Bash 语法）。

---

## 4. 初始配置

### 4.1 注册到 AI Agent（一条命令）🟢

```bash
rea setup
```

交互式向导会：把 REA 的 MCP 服务器写入你选择的 Agent 配置（自动备份原配置）、安装配套的工作流说明（skill）。完成后**重启 Agent** 才能看到工具。

非交互批处理（适合脚本/CI）：

```bash
rea setup --all-detected --yes        # 为检测到的所有 Agent 注册
rea setup --dry-run                   # 只打印将要做的修改，不落盘
rea setup --client <客户端ID> --yes    # 只注册指定客户端（ID 可见 --dry-run 输出）
```

### 4.2 手动注册 MCP（任何支持 MCP 的客户端）🟡

在客户端配置中加入：

```json
{
  "mcpServers": {
    "rea": {
      "command": "npx",
      "args": ["-y", "rea-agents@6.5.0", "mcp"]
    }
  }
}
```

> 生产使用建议锁定确切版本号（如上），避免 `@latest` 意外升级。

### 4.3 可选：配置深度分析引擎（只影响原生二进制分析）🟡

| 引擎           | 配置方式                                                          |
| -------------- | ----------------------------------------------------------------- |
| Ghidra         | 设置环境变量 `GHIDRA_INSTALL_DIR` 指向解压后的 Ghidra 12.1.x 目录 |
| Hopper         | 设置 `HOPPER_LAUNCHER_PATH`，或让 `rea setup` 安装                |
| IDA            | 设置 `REA_IDA_MCP_CONFIG` 指向 IDA MCP 注册 JSON                  |
| JADX（APK 用） | 设置 `REA_JADX_MCP_JAR` 指向 jadx-headless-mcp 的 JAR             |

不做这一步也完全没问题——JS/Electron、.NET 静态分析不需要任何引擎。

### 4.4 自检：确认环境就绪 🟢

```bash
rea doctor
```

输出是一份分项体检表：`node`、`host` 健康即可放心使用；`hopper/ghidra/ida/jadx` 显示 `missing_analysis_engine` 表示该能力未配置（不影响其他功能）；`config_drift` 表示某个 Agent 注册需要刷新，重跑 `rea setup` 即可。

---

## 5. 基础操作：5 分钟上手

### 5.1 看帮助 🟢

```bash
rea --help             # 命令总览
rea <命令> --help      # 某个命令的详细参数
```

### 5.2 环境体检 🟢

```bash
rea doctor                    # 全量体检
rea doctor --json             # 机器可读输出（脚本用）
rea doctor --target "D:/apps/example"   # 针对某个目标检查
```

### 5.3 分析一个应用（通用入口）🟢

```bash
rea analyze "D:/apps/example"
```

`analyze` 会先识别目标类型（原生程序、.NET、JS 应用、分析数据库等），给出整体概览，再按需深入。

### 5.4 静态分析 JavaScript / Electron 应用 🟢

```bash
rea analyze-javascript-application "D:/apps/example" --json > result.json
```

结果包含：模块清单、import/依赖图、Electron 主进程/渲染进程边界、预加载脚本与原生插件关系，每项结论都带证据来源。

### 5.5 输出格式 🟡

所有命令都支持统一的输出选项：

```bash
--format json|yaml|md|jsonl|toon   # 选择输出格式
--filter-output <键路径>            # 只保留关心的字段，如 foo,bar.baz,a[0,3]
--full-output                      # 显示完整信封（默认可能省略部分信息）
```

### 5.6 常用命令速查 🟢

| 命令                                                | 作用                         |
| --------------------------------------------------- | ---------------------------- |
| `rea doctor`                                        | 环境自检，部署后第一件事     |
| `rea analyze <路径>`                                | 通用分析入口，先看整体       |
| `rea analyze-javascript-application <路径>`         | JS/Electron 静态结构分析     |
| `rea inspect-managed-artifact <dll>`                | .NET 元数据/CIL 检查         |
| `rea inspect-managed-members <dll> --type <类型名>` | 查看某个 .NET 类型的成员     |
| `rea decompile <参数>`                              | 反编译目标的一部分（需引擎） |
| `rea update`                                        | 更新全局安装                 |
| `rea uninstall [--purge-data]`                      | 移除 Agent 注册与技能文件    |

---

## 6. 进阶用法

### 6.1 在 AI Agent 里使用 🟡

注册成功后（第 4 节），直接对 Agent 说人话即可，例如：

```text
用 REA 分析 D:\apps\example 这个 Electron 应用：
它的主进程和渲染进程分别做了什么？剪贴板功能是怎么实现的？给出证据。
```

Agent 会自动调用 REA 的工具（打开目标 → 追踪代码 → 导出证据），并把结论和证据一起交给你。工具返回的字符串、反编译代码等内容一律视为**数据**而非指令——REA 会在结果中标记不可信来源，防止目标文件里藏提示词注入。

### 6.2 证据（Evidence）与快照 🟡

每次分析产出的是一个 **Evidence 包**：谁（工具/引擎）、对什么（目标+校验摘要）、做了什么（操作+参数）、得到什么（结果），全部可复核。你可以：

- 保存为 JSON 留档或贴进报告；
- 用 `rea compare` 比较两次分析的差异（比如软件升级前后）；
- 用 `--snapshot` 复用本地分析快照，避免重复劳动。

### 6.3 脚本化与自动化 🔴

```bash
# 批量分析多个应用并汇总
for app in D:/apps/*; do
  rea analyze-javascript-application "$app" --json --filter-output normalized_result > "$(basename "$app").json"
done

# 检查关键字段是否为空（退出码可用于 CI）
rea doctor --json | jq -e '.environment_healthy' > /dev/null
```

退出码约定：`0` 成功；非 `0` 表示失败，具体状态见 [docs/cli.md](docs/cli.md)。

### 6.4 值得知道的进阶开关 🔴

- `--provider <id>`：指定或自动选择深度分析引擎；
- `--integrity-policy fail|record-and-continue`：目标文件与声明摘要不一致时的行为；
- `--max-heap-mb` / `--analysis-timeout-ms`：大项目分析的内存与超时预算；
- `REA_GHIDRA_STARTUP_TIMEOUT_MS` / `REA_HOPPER_STARTUP_TIMEOUT_MS`：大二进制首次分析时放宽引擎启动时限。

---

## 7. 实战示例

### 例 1：分析一个 Electron 应用（无需任何分析引擎）🟢

```bash
rea analyze-javascript-application "D:/apps/example" --json > app.json
```

输出里重点看三块：

1. `normalized_result.graph.nodes`：模块与文件节点（谁引用了谁）；
2. Electron 边界标记：`electron-main` / `electron-renderer` 等角色，帮你快速定位主进程逻辑；
3. `integrity_contradictions`：目标在分析期间是否被改动（空数组 = 一致性良好）。

### 例 2：检查 .NET 程序集 🟢

```bash
rea inspect-managed-artifact "D:/apps/example/Assembly.dll" --json
rea inspect-managed-members "D:/apps/example/Assembly.dll" --type Example.Parser --json
```

第一条给出程序集元数据与依赖；第二条列出指定类型的成员（含方法签名），适合快速搞清一个类库的公开接口。

### 例 3：部署后自检（推荐照做）🟡

```bash
rea doctor                    # 1) 环境体检
rea mcp doctor --json         # 2) MCP 服务器启动自检（含子进程日志）
```

还可以手动验证 MCP 服务器能否握手（返回 `serverInfo` 即正常）：

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"check","version":"1.0"}}}' | npx -y rea-agents@6.5.0 mcp
```

---

## 8. 常见问题排查（FAQ）

**Q1：运行时报 Node 版本错误 🟢**
按 `package.json#engines` 要求升级到 22.19+ / 24.11+ / 26+。用 nvm 的话：`nvm install 24 && nvm use 24`。注意本项目 `.npmrc` 开启了 `engine-strict=true`，版本不符会直接拒绝安装。

**Q2：Windows 源码构建提示 "REA Windows native controls are unavailable" 🟡**
源码检出缺少 `native/windows/build/manifest.json` 和 `rea-windows-x64.node`（这两个文件只能在 Linux+MinGW 构建）。按第 3 节方式 C 的命令从 `npm pack rea-agents@6.5.0` 的 tarball 中恢复即可，构建随即通过。

**Q3：`git clone` 报 SSL/TLS handshake 失败 🟢**
多为瞬时网络问题或代理配置问题。先重试；仍失败时检查 `https_proxy` 环境变量，或改用 `git config --global http.proxy http://127.0.0.1:<端口>`。

**Q4：`npx` / `npm install` 很慢 🟡**
国内网络可临时切换镜像：`npm install --registry=https://registry.npmmirror.com`。npx 首次运行会下载整包，耐心等待或用方式 B 全局安装。

**Q5：Agent 里看不到 REA 的工具 🟡**
① 确认 `rea setup` 已完成且选择了该客户端；② **完全重启** Agent（不是刷新窗口）；③ 用 `rea doctor` 看对应 `registration:*` 是否为 `config_drift`；④ 手动检查客户端配置里是否有 `mcpServers.rea`。

**Q6：`rea doctor` 显示 hopper/ghidra/ida 缺失，是坏了吗？🟢**
不是。这只表示"原生二进制深度分析"未配置。JS/Electron、.NET、网站等分析不受影响；需要时再按 4.3 配置任一引擎即可。

**Q7：分析原生 exe 时提示找不到分析引擎 🟡**
REA 不自带反编译器，需要 Hopper / Ghidra / IDA 之一。Ghidra 免费：下载解压后设置 `GHIDRA_INSTALL_DIR` 环境变量，再用 `rea doctor` 确认变为健康。

**Q8：Linux 上 Ghidra/JADX/固件工具启动失败 🔴**
需要系统有 procps 兼容的 `ps`（REA 用它做进程归属校验）。安装 `procps` 后重试，详见 [docs/installation.md](docs/installation.md)。

**Q9：分析大文件很慢或超时 🔴**
调大预算：`--analysis-timeout-ms`、`--max-heap-mb`，以及引擎启动时限环境变量（见 6.4）。Ghidra 首次分析大二进制需要建立项目，属正常现象。

**Q10：怎么升级 / 彻底卸载？🟡**
升级：全局安装用 `rea update`；npx 方式用 `npx rea-agents@latest`。卸载：`rea uninstall` 只移除 Agent 注册与技能文件；`rea uninstall --purge-data` 连同 `~/.rea` 缓存与状态一起清理。源码方式则 `git pull && npm ci && npm run build:cached`。

**Q11：杀毒软件报警 / 文件被拦截？🟢**
REA 会启动 Ghidra、调试器等外部工具，可能被启发式规则误报。请将安装目录加入信任区；企业环境请遵循本单位安全规定。

---

## 9. 目录结构与技术栈

| 路径       | 内容                                                      |
| ---------- | --------------------------------------------------------- |
| `src/`     | TypeScript 源码（领域层、应用层、适配器、MCP/CLI 组合层） |
| `bridge/`  | 与 Hopper/Ghidra/IDA/JADX 等外部引擎的桥接脚本            |
| `native/`  | 平台原生组件（Windows 控件、Swift 辅助等）                |
| `scripts/` | 构建、校验、打包与维护脚本                                |
| `docs/`    | 完整文档（安装、CLI、MCP 契约、各分析场景指南）           |
| `tests/`   | 单元、契约、边界与验收测试                                |

技术栈：Node.js 22+/24+ · TypeScript · MCP（stdio）· Turbo 构建 · Vitest 测试。构建入口：`npm run build:cached`；测试：`npm test`；全量检查：`npm run check`。

---

## 10. 许可与合规

- 本项目基于 [morluto/rea](https://github.com/morluto/rea)，采用 **MIT** 许可，见 [LICENSE](LICENSE)。
- 第三方组件与外部工具（Ghidra、Hopper、IDA、JADX 等）各有其许可与使用条款，见 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)。商业工具（IDA Pro、Hopper、JEB）**必须自行合法购买并授权**，本项目不提供任何许可或激活。
- 仅将 REA 用于合法研究、学习、互操作与软件保存；请勿用于未授权分析或规避软件许可。完整声明见 [DISCLAIMER.md](DISCLAIMER.md)。
- 请从官方渠道获取本项目：[GitHub 仓库](https://github.com/morluto/rea) / [rea.tools](https://rea.tools/) / [npm 包](https://www.npmjs.com/package/rea-agents)。相似名称的仓库或复制来的 README 不代表其为官方发布。
