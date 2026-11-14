# Agent-Kernel-Toolbox

**减少截图与视觉推理开销，致力于大幅降低 AI 软件及网页交互的 Token 消耗和长期使用成本。**

**Agent-Kernel-Toolbox** 以**工具包 + MCP + Skills** 的形式整合浏览器、应用与程序分析能力。通过结构化读取网页 DOM/ARIA、原生控件和程序运行信息，辅助 AI 更高效、深入地理解软件，减少反复截图、视觉识别与试探式操作带来的开销。项目面向能够调用本地工具、控制计算机的 AI 客户端；访问在线网页还需网络连接，**不适用于无法调用外部工具的纯对话模式**。

`Python 3.10+` · `MCP stdio` · `Skills` · `PANL 1.0` · `按配置授权工具和目标`

> **版本核对（请在发布前阅读）**  
> 本文档对应的归档文件名为 `Agent-Kernel-Toolbox-0.5.6.3.zip`，但归档内 `runtime-manifest.json`、MCP 文档、浏览器扩展、构建元数据和 Python 运行时声明的实际版本均为 **`0.5.6.2`**。包内检查也返回 `0.5.6.2`。因此，下文描述的是**当前归档已包含的功能**，不能据此认定已完成 `0.5.6.3` 升级；本文件不虚构 `0.5.6.3` 更新日志。正式以 `0.5.6.3` 发布前，需统一版本标识、重建发行包并重新验证。

## 项目概览

Agent-Kernel-Toolbox 将不同系统和分析后端封装为固定范围的 MCP 工具。工具的可见性和可操作对象由**本地安装者的配置文件**决定，包括获准工具、进程、设备、站点来源、输入文件、编译器和执行权限。MCP 客户端不能在单次请求中临时扩大授权范围。

**成本说明：** 浏览器组件确实提供无需截图/OCR 的 DOM/ARIA 与文字读取途径，但当前发行包未附与纯视觉交互方案直接比较的 Token 或费用基准。“大幅降低”是项目的优化目标，实际节省幅度取决于任务、模型、页面结构和工具调用方式；复杂自绘界面等场景可能仍需要视觉能力。

当前归档的可核对信息：

| 项目 | 当前包内信息 |
| --- | --- |
| 包内实际版本 | `0.5.6.2`（归档文件名标记为 `0.5.6.3`） |
| MCP 传输 | 本地子进程 `stdio`，逐行 JSON-RPC 2.0 |
| MCP 协议版本 | `2025-11-25`、`2025-06-18` |
| 工具目录 | **162 项工具、36 个工具族** |
| Python 实现 | `runtime-access.pyz`，内含 49 个 Python 源文件 |
| 附带示例 | 13 个跨平台可重定位对象文件及清单 |
| 原生组件 | C、Rust、Go、Java 源码与按需构建入口 |
| 项目许可证 | [Permissive Attribution and Notice License (PANL) 1.0](LICENSE)；包内自有组件的许可标记已同步更新，第三方依赖仍适用各自许可证 |

**能力边界：** 工具存在于目录中，只代表代码提供了相应入口；实际调用还依赖操作系统、目标程序、设备、配置权限及外部后端。安装检查通过也不表示所有功能都在当前设备上完成了实测。

## 快速开始

### 1. 环境准备与解压

基础 MCP Host 需要 **Python 3.10 或更高版本**。当前实现主要面向 **Linux / POSIX 宿主环境**，核心运行时使用 Python 标准库；浏览器、编译器、JVM 和分析引擎仅在用到对应功能时安装。**允许在其他系统中尝试运行或自行适配，但不保证兼容性**，尤其是依赖 Linux `/proc`、pidfd、`fcntl`、AT-SPI 等接口的功能。具体的已测试系统、检查范围与未经验证的平台，见下方「系统兼容性与实测范围」。

```bash
unzip Agent-Kernel-Toolbox-0.5.6.3.zip -d Agent-Kernel-Toolbox
cd Agent-Kernel-Toolbox
python3 --version
```

先校验原始 ZIP 的内容一致性（示例路径须替换为 ZIP 实际位置）：

```bash
python3 verify-runtime.py /absolute/path/Agent-Kernel-Toolbox-0.5.6.3.zip
```

如通过可信发布渠道**独立取得**整个 ZIP 的 SHA-256，可进一步固定预期摘要：

```bash
python3 verify-runtime.py /absolute/path/Agent-Kernel-Toolbox-0.5.6.3.zip \
  --expected-sha256 EXPECTED_SHA256
```

`CONTENTS.json` 与校验脚本用于检查文件一致性；**包内提供的哈希值不等于发布者身份认证**。

### 2. 检查安装及 MCP 连接

```bash
python3 tools/check_installation.py
python3 tools/check_installation.py --mcp
```

第一条进行离线安装与工具目录检查；第二条额外建立**临时、只读**的 stdio MCP 连接，并检查两种受支持的协议版本。成功结果包含 `passed: true`、`version`、`tools` 和 `mcp_protocols` 等字段。

这两项检查不会自动启动目标应用、浏览器或分析引擎；具体后端须分别验证。

### 3. 创建私有配置

在 Toolbox 根目录执行：

```bash
mkdir -p "$HOME/.config/agent-kernel-toolbox" "$HOME/.local/state/agent-kernel-toolbox"
chmod 700 "$HOME/.config/agent-kernel-toolbox" "$HOME/.local/state/agent-kernel-toolbox"
cp runtime-minimal-config.example.json "$HOME/.config/agent-kernel-toolbox/config.json"
chmod 600 "$HOME/.config/agent-kernel-toolbox/config.json"
```

编辑刚复制的 `config.json`，将 `state_directory` 替换成当前用户拥有的**真实绝对路径**。最小配置结构如下：

```json
{
  "schema_version": 1,
  "state_directory": "/home/YOUR_USER/.local/state/agent-kernel-toolbox",
  "allowed_tools": [
    "runtime.status",
    "runtime.capabilities",
    "runtime.audit",
    "build.inspect"
  ],
  "portable_build": {
    "profiles": {}
  }
}
```

注意：

- `/home/YOUR_USER/...` 只是示例，必须替换；JSON 中的 `~` 和环境变量不会自动展开。
- 配置文件应为当前用户拥有的普通文件，不能是符号链接，也不能允许组或其他用户写入；建议权限为 `0600`。
- 状态目录须为当前用户独占的私有目录（建议 `0700`），一份固定配置对应一个独立状态目录，同一目录只允许一个活动 Host。
- 初次连接只保留最小只读工具；完成目标、权限与后端核对后，再按任务增加 `allowed_tools`。

### 4. 接入 MCP 客户端

按客户端要求配置本地 MCP 服务，参考 [mcp-client.example.json](mcp-client.example.json)：

```json
{
  "mcpServers": {
    "Agent-Kernel-Toolbox": {
      "command": "/absolute/path/to/python3",
      "args": [
        "/absolute/path/Agent-Kernel-Toolbox/runtime-access.pyz",
        "serve",
        "--config",
        "/absolute/path/to/private/config.json"
      ]
    }
  }
}
```

将 `command`、pyz 路径及配置路径替换为实际绝对路径。不同 MCP 客户端的配置入口可能不同，但启动命令等价于：

```bash
python3 runtime-access.pyz serve --config /absolute/path/to/private/config.json
```

服务启动后等待 stdin 的 MCP 消息，**不会打开网页界面**。标准输出仅用于 JSON-RPC 响应，诊断信息写入标准错误。客户端依次执行 `initialize`、`notifications/initialized`、`tools/list`，然后调用获准的 `tools/call`；结束时关闭 stdin 并等待子进程退出。

例如，调用只读状态工具：

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "tools/call",
  "params": {
    "name": "runtime.status",
    "arguments": {}
  }
}
```

协议说明和完整握手示例见 [MCP.md](MCP.md)。`tools/list` **只返回已授权工具**及它们当前的 `inputSchema`，应以运行时返回的参数定义为准。

## 能力范围

| 能力 | 主要工具族 | 前提与限制 |
| --- | --- | --- |
| 状态、能力、审计和策略 | `runtime.*`、`policy.*` | 固定的 Host 配置和私有状态目录 |
| 浏览器导航、DOM/ARIA、表单及网络控制 | `browser.*` | Node.js、Playwright、实际浏览器及明确允许的 origins；输入操作需单独授权 |
| Linux 进程、内存、CPU 与原生控件 | `process.*`、`native.*` | 真实进程身份、相应权限；原生 UI 需 AT-SPI |
| 跨平台应用和调试 | `appium.*`、`debugger.*` | 已登记的 Appium/LLDB、设备及应用 profile |
| 文件、程序入口与逆向分析 | `binary.*`、`app.*`、`reverse.*` | 已登记的输入文件及指定分析后端 |
| 固定 IR 编译与受限转换 | `build.*`、`ir.*`、`translate_native.*` | 固定摘要的输入、编译器、目标及显式编译许可 |
| 结构、符号分析和仿真 | `analysis.*`、`symbolic.*`、`structure.*`、`portable.*` | 已配置的外部引擎、输入范围和资源预算 |
| 程序、指令和环境观察 | `probe.*`、`program.*`、`metadata.*`、`environment.*` | 预先登记的目标和观察范围 |
| 协作式状态与 JVM | `coordinated.*`、`semantic.*`、`managed.*`、`managed_probe.*` | 已接入的协作代理或已登记的 JVM/JDI/JVMTI 环境 |
| 进程树、文件活动、指标与 HTTP | `tree.*`、`files.*`、`malware.*`、`network.*` | 固定目标、规则、请求范围和所需内核权限 |
| 跟踪、回放与兼容性分析 | `trace.*`、`debug_native.*`、`compatibility.*` | 固定 GDB/Frida 等后端及目标系统权限 |
| QEMU 整机运行状态 | `machine.*` | 固定的机器模型、镜像、固件/内核与 QEMU |
| CPU 探测、执行预算与图形过滤 | `cpu.*`、`probes.*`、`render.*` | 硬件/内核条件；OpenGL 过滤需专用预加载组件 |

建议每次任务遵循 **确认授权 → 读取当前状态 → 操作 → 独立验证结果 → 释放资源** 的顺序。进程身份、页面节点、对象句柄等应根据最新观察结果使用，不能在导航、重启或目标变更后盲目复用。

## 系统兼容性与实测范围

项目不在许可证层面对操作系统作限制；**其他 Linux 发行版、macOS、Windows 等环境均可自行尝试部署或适配**。不过，软件能否启动、能使用哪些工具、能否操作特定应用，取决于宿主 API、Python 版本、运行依赖、设备驱动和权限。下表区分**本次实际执行的检查**与**尚未进行的系统级验证**，避免把“代码中存在适配器”误读为“对应平台已测试通过”。

| 系统 / 环境 | 验证情况 | 说明 |
| --- | --- | --- |
| **Debian GNU/Linux 13 (trixie)，x86_64，Python 3.13.5** | **已通过部分验证** | 本次在该环境执行 `tools/check_package_contract.py`、`tools/check_installation.py --mcp`；离线包契约、包体一致性、工具目录和 MCP `2025-11-25` / `2025-06-18` 的只读握手检查通过 |
| 其他 Linux 发行版 / CPU 架构 | **尚未逐一实测** | 可尝试安装；需满足 Python、系统库和功能对应的内核 / CPU 条件，可能需要额外适配 |
| macOS / 其他 POSIX / Unix 类系统 | **本地 MCP Host 尚未验证** | 可自行测试或适配；部分 Linux 专用进程、控件及权限接口未必存在。通过远程 Appium / LLDB 连接 macOS 目标，不等于本地 Host 已获验证 |
| Windows（原生环境） | **尚未验证，不保证直接运行** | 当前宿主包含 POSIX 文件锁和 Linux 专用能力；可以考虑 WSL 或自行移植，但 WSL 部署也未在本次测试中验证。远程 Windows Appium 适配器与原生 Host 是不同部署方式 |
| Android / iOS | **设备端实测未确认** | 主要作为已配置 Appium / 调试后端的目标设备接入；不视为本地 MCP Host 的已支持安装平台 |

**测试边界：** 本次确认的是离线发行包检查和两种 MCP 协议的**临时只读握手**，不包含全量 Python 回归、真实浏览器自动化、进程写入、GUI 控件、设备 Appium、LLDB、跨架构目标机运行、可选分析引擎或生产负载验证。已通过的检查也**不能外推为所有功能或所有系统兼容**。代码及工具声明覆盖的格式、CPU、对象文件和远程目标，均须在具体运行环境下单独验收。

参考：[COMPATIBILITY.md](COMPATIBILITY.md)（能力与测试边界）、[CROSS_PLATFORM.md](CROSS_PLATFORM.md)（跨系统目标接入与已知限制）。

## 可选环境与组件

以下是**按功能需要**安装的组件，不表示这些组合均已实测：

| 场景 | 依赖与条件 |
| --- | --- |
| 基础 MCP Host | Python 3.10+；当前主要面向 Linux / POSIX，其他系统兼容性不保证 |
| Playwright 浏览器 | Node.js **24+**、`browser/package-lock.json` 固定的 Playwright、可运行的浏览器 |
| 固定 word-IR 编译 | POSIX 环境及已登记、固定摘要的 GCC / Clang / Zig；编译器需支持目标 triple |
| 本机 Linux UI / 进程操作 | procfs/pidfd、D-Bus、AT-SPI、ptrace 或 perf 等对应权限（按功能需要） |
| C / Rust / Go 原生辅助组件 | 匹配宿主 ABI 的工具链；Rust 控制器见自身 `Cargo.toml` |
| JVM / 管理型探针 | JDK/JVM、固定 classpath；探针构建可能需要 JVMTI |
| 可选分析与检测后端 | 根据目标安装 pyelftools、Z3、angr、Frida、YARA、Ghidra、JADX、GDB、QEMU、strace 等 |

**可选分析引擎不随工具包附带。** Windows、macOS、Android、iOS 相关工具可通过实际配置的远程设备/调试适配器与目标交互；这并不证明所有平台都能直接运行本地 Host。运行前请先完成最小只读验证，再针对所需功能做独立的实际测试。

## 常用工作流

### 浏览器自动化

安装当前包锁定的 Node 依赖；需要 Playwright 管理的 Chromium 时再安装浏览器：

```bash
cd browser
npm ci --ignore-scripts --omit=dev --no-audit --no-fund
npx playwright install chromium
cd ..
```

按 [runtime-config.example.json](runtime-config.example.json) 和 [browser-forms-config.example.json](browser-forms-config.example.json) 设置浏览器路径、精确的 `allowedOrigins`，并单独决定是否开放编辑文本读取及表单输入。

典型顺序：`browser.open → browser.snapshot / browser.query → browser.set_text / browser.invoke → 重新观察`。修改页面前应取得当前 `snapshot_id`、`node_id`，导航或文档替换后重新生成句柄。独立浏览器扩展、流量规则与诊断说明见 [browser/README.md](browser/README.md)。

### 进程与原生 UI

启动新程序时，在配置的 `profiles` 中登记固定命令，再由 `process.launch` 选择对应 profile；不能从一次工具调用中提供未经登记的任意命令。

附加到正在运行的进程前，先获取其当前身份：

```bash
python3 runtime-access.pyz identify --pid 12345
```

将**真实进程身份**登记到 `targets` 后再调用 `process.attach`。PID 可能复用；内存写入、调试、进程控制与原生 UI 输入分别需要明确授权。Linux 控件操作通常按 `native.open → native.snapshot/query → native.focus/set_text/invoke → 重新观察` 进行。参阅 [NATIVE.md](NATIVE.md)、[RUNTIME.md](RUNTIME.md)。

### 检测平台并编译固定 word-IR

随包提供 [portable-word-ir.example.json](portable-word-ir.example.json) 和独立的 CLI。首先检测平台：

```bash
python3 tools/runtime-platform.py detect
```

参考 [portable-build-config.example.json](portable-build-config.example.json) 创建自己的编译配置，填写输入文件、编译器的**绝对路径与 SHA-256**，以及目标架构。保持 `allow_compile: false` 先查看计划：

```bash
python3 tools/runtime-platform.py inspect \
  --config /absolute/private/build.json \
  --state-directory /absolute/private/build-state

python3 tools/runtime-platform.py plan \
  --config /absolute/private/build.json \
  --state-directory /absolute/private/build-state \
  --profile local
```

确认计划后，将该 profile 的 `allow_compile` 设为 `true`，再使用**新的** `request_id` 编译：

```bash
python3 tools/runtime-platform.py compile \
  --config /absolute/private/build.json \
  --state-directory /absolute/private/build-state \
  --profile local \
  --request-id build-local-001
```

该路径主要把受限的纯整数 word-IR 编译为**可重定位对象文件**，并检查对象格式、架构、端序和 ABI。后续链接、签名、运行环境与目标机执行必须另外完成；`target_execution_verified: false` 不代表目标已运行。

对象样例与支持的目标见 [PORTABLE-BUILD.md](PORTABLE-BUILD.md) 和 [examples/portable-objects/README.md](examples/portable-objects/README.md)。

### 原生 SDK 与协作代理

在具备相应 C11、Rust 工具链的环境中，可构建随包的原生示例：

```bash
python3 native/runtime-agent/build.py \
  --output-directory /absolute/native-agent-build
```

该示例包含可接入自有程序的 C 代理、Rust 控制器、示范程序和对象布局元数据；它**不会自动为未接入代理的程序提供对象恢复**。详见 [native/runtime-agent/README.md](native/runtime-agent/README.md)、[native/README.md](native/README.md)。

## 配置模板索引

配置文件必须根据实际部署环境修改，模板中占位路径、全零摘要和设备标识不能直接用于生产。按需选用配置，**不要把多个模板的 `allowed_tools` 不加审查地合并**。

| 使用场景 | 模板 |
| --- | --- |
| 最小状态与 MCP 连接 | [runtime-minimal-config.example.json](runtime-minimal-config.example.json) |
| 浏览器与表单输入 | [runtime-config.example.json](runtime-config.example.json)、[browser-forms-config.example.json](browser-forms-config.example.json) |
| Linux 原生进程与浏览器辅助 | [native-config.example.json](native-config.example.json)、[native-browser-config.example.json](native-browser-config.example.json) |
| 编译型原生 worker | [compiled-native-config.example.json](compiled-native-config.example.json) |
| 远程设备、应用与调试 | [cross-platform-config.example.json](cross-platform-config.example.json)、[debugger-profile.example.json](debugger-profile.example.json) |
| 目标程序与环境观察 | [monitoring-config.example.json](monitoring-config.example.json) |
| 进程树、文件活动、规则与网络 | [defensive-config.example.json](defensive-config.example.json) |
| 逆向与外部分析后端 | [reverse-config.example.json](reverse-config.example.json) |
| 高级分析、JVM、整机与 IR | [runtime-advanced-config.example.json](runtime-advanced-config.example.json) |
| 独立编译 CLI | [portable-build-config.example.json](portable-build-config.example.json) |

`v055-config.example.json` 是高级配置的兼容文件名，内容与 `runtime-advanced-config.example.json` 对应。

## 安全、审计与异常恢复

对可能改变目标的工具调用，使用独立且不可复用的 `request_id`。它与 JSON-RPC 的请求 `id` 用途不同：前者记录特定操作的持久状态，后者用于匹配协议响应。

除 JSON-RPC `error` 外，客户端还需检查 MCP `isError`、`structuredContent.ok`，以及存在时的 `unknown_effect`、`effect_unknown`、`effect_dispatched` 字段。**请求超时或效果 Unknown 时，不得简单换一个 `request_id` 重新尝试。** 应停止对相关目标的进一步修改，保留状态目录，并以只读观察确认实际效果。

停止使用该状态目录的 Host 后，可以进行离线审计：

```bash
python3 runtime-access.pyz audit \
  --state-directory /absolute/private/toolbox-state
```

由操作者核对实际结果后，才能记录处置，例如：

```bash
python3 runtime-access.pyz reconcile \
  --state-directory /absolute/private/toolbox-state \
  --request-id operation-001 \
  --disposition effect_observed
```

允许的处置值有 `effect_observed`、`no_effect_observed`、`abandoned`。**对账只记录结论，不会重放、撤销或回滚原操作。** 具体生命周期约束见 [LIFECYCLE.md](LIFECYCLE.md) 与 [MCP.md](MCP.md)。

## 测试与诊断

在工具包根目录运行：

```bash
# 离线检查发行包契约
python3 tools/check_package_contract.py

# 完整性检查 + 两种 MCP 协议的临时只读握手
python3 tools/check_installation.py --mcp

# Python 回归测试（其中部分测试需可选依赖和宿主权限）
python3 -m unittest discover -s tests -v
```

需要第三方验证依赖时可在独立 Python 环境中使用 [requirements-validation.txt](requirements-validation.txt)。浏览器测试在安装依赖后运行：

```bash
cd browser
npm run test:unit
npm run test:browser
```

更深入的工具协议、桌面、跨架构或后端检查请查阅 [LIVE_CHECKS.md](LIVE_CHECKS.md)、[ADVANCED_CHECKS.md](ADVANCED_CHECKS.md)、[MCP.md](MCP.md)。这些检查可能启动测试夹具、编译代码或访问已配置后端，先阅读命令 `--help` 和对应指南。

**本次明确记录的实测环境：** Debian GNU/Linux 13 (trixie)、x86_64、Python 3.13.5。`tools/check_package_contract.py` 与 `tools/check_installation.py --mcp` 均返回 `passed: true`；MCP `2025-11-25`、`2025-06-18` 只读握手各自通过。**其他系统及浏览器、内核跟踪、远程设备、可选分析引擎的端到端操作未据此获验证**，详见上方「系统兼容性与实测范围」。

## 源码、重建与发布检查

Python 源码打包在 `runtime-access.pyz` 中。如需查看或修改源文件，可提取到**尚不存在**的目录：

```bash
python3 tools/extract_sources.py --output src
```

使用 `build-source.py` 或 `build-runtime.py` 可根据固定资源清单重新构建；输出 ZIP 路径须尚不存在：

```bash
python3 build-source.py --output /absolute/new/Agent-Kernel-Toolbox-rebuilt.zip
python3 verify-runtime.py /absolute/new/Agent-Kernel-Toolbox-rebuilt.zip
```

`build-runtime.py` 维护显式打包清单；构建过程会重新生成 `runtime-access.pyz` 和 `CONTENTS.json`。**替换发行包中的 README.md 或 LICENSE，会让旧 `CONTENTS.json` 的文件摘要失效**。因此，更新文档或许可证后必须重新打包，并对新包重复安装与一致性检查；不要通过关闭校验来处理差异。**本次更新已同步包内 README.md、LICENSE、项目组件元数据及 CONTENTS.json，并执行发行包一致性检查。**

如确实要发布 `0.5.6.3`，还必须先确认该版本的真实改动，并同步运行时版本、`runtime-manifest.json`、MCP 文档、浏览器扩展、npm/Cargo 元数据及其他发布标识。**修改 ZIP 文件名或 README 标题不会升级二进制版本。** 本次已更新包内自有组件的许可声明；**第三方依赖（包括可选的 `@jshookmcp/jshook`）可能仍受 AGPL 或其他许可证约束，不得删除其许可信息**。许可证变更还须由拥有相应版权和再许可权的主体确认；发布流程见 [PACKAGING.md](PACKAGING.md)。

## 目录与文档

```text
Agent-Kernel-Toolbox/
├── runtime-access.pyz            # Python 运行时 / MCP 入口
├── runtime-manifest.json         # 运行时身份、能力与打包元数据
├── LICENSE                       # PANL 1.0 许可证
├── CONTENTS.json                 # 包文件 / 内嵌 Python 摘要清单
├── verify-runtime.py             # ZIP / 目录完整性验证
├── mcp-client.example.json       # MCP 客户端配置示例
├── *config.example.json          # 不同任务的授权配置模板
├── browser/                     # Playwright 适配器及扩展
├── native/                      # C / Rust / Go / JVM 原生组件
├── examples/portable-objects/   # 13 个目标对象及元数据
├── tools/                       # 检查、平台与构建辅助入口
├── tests/                       # 回归测试与测试夹具
├── skills/agent-kernel-toolbox/ # 本地 Skill 与只读检查器
├── build-source.py              # 兼容重建入口
└── build-runtime.py             # 固定资源清单构建器
```

按任务查阅专题文档：

- **接入与运行：** [MCP.md](MCP.md)、[RUNTIME.md](RUNTIME.md)、[LIFECYCLE.md](LIFECYCLE.md)
- **浏览器及原生操作：** [browser/README.md](browser/README.md)、[NATIVE.md](NATIVE.md)、[NATIVE_WORKER.md](NATIVE_WORKER.md)
- **平台与构建：** [CROSS_PLATFORM.md](CROSS_PLATFORM.md)、[COMPATIBILITY.md](COMPATIBILITY.md)、[PORTABLE-BUILD.md](PORTABLE-BUILD.md)、[PACKAGING.md](PACKAGING.md)
- **逆向、监测与防御性观察：** [REVERSE.md](REVERSE.md)、[PROGRAM-MONITORING.md](PROGRAM-MONITORING.md)、[DEFENSIVE-OBSERVATION.md](DEFENSIVE-OBSERVATION.md)
- **客户端 Skill：** [skills/agent-kernel-toolbox/SKILL.md](skills/agent-kernel-toolbox/SKILL.md)

## 常见问题

| 现象 | 排查方向 |
| --- | --- |
| 启动后没有可见窗口 | `serve` 是 stdio MCP 服务；用 MCP 客户端接入，或先运行 `tools/check_installation.py --mcp` |
| 工具没有显示在 `tools/list` | 检查 `allowed_tools` 是否包含该工具，并重启对应 Host |
| `config_must_be_owned_regular_file` | 检查配置文件所有者、普通文件类型、符号链接及写权限 |
| `state_directory_must_be_private` | 使用当前用户独占、权限 `0700` 的私有目录 |
| 状态目录锁冲突或配置摘要不匹配 | 检查是否存在活动 Host；先处理现有状态和未决操作 |
| 浏览器 origin、句柄被拒绝 | 核对配置的精确 origin；在当前页面重新获取快照和句柄 |
| 输入文件或编译器 SHA-256 不匹配 | 核对实际文件、路径与可信摘要，不跳过校验 |
| ptrace、AT-SPI、perf 或浏览器启动失败 | 核对当前用户权限、系统服务、设备和所需依赖 |
| 操作结果为 Unknown | 保留状态并确认实际效果，停止盲目重试；必要时离线对账 |

## 许可证

本包根目录提供 [**Permissive Attribution and Notice License (PANL) 1.0**](LICENSE)。完整、具有约束力的英文条款请以同目录 `LICENSE` 为准。以下是便于阅读的中文概要，不替代许可证原文：

- **允许的使用方式：** 免费使用、复制、修改、合并、发布、分发、再许可和销售；允许商业用途和专有/闭源项目，无需公开衍生代码。
- **署名与保留：** 在副本和衍生作品中保留原版权声明、作者/组织署名和 PANL 1.0 全文；闭源发行应在随附文档、法律声明或可访问的许可证/关于页面展示署名。
- **事先邮件通知：** 在**商业使用**或**纳入专有/闭源项目**之前，向 **[wuOO364@gmail.com](mailto:wuOO364@gmail.com)** 发送邮件，说明 (a) 项目/产品与责任主体、(b) 预期用途（包括商业或闭源用途）、(c) 明确排除的用途。**发送邮件后即可开始使用，无需等待批准、确认或回复。** 已通知的项目后续版本和衍生作品，在预期用途没有实质变化时无需重复通知。
- **衍生作品：** 可以采用其他许可证，包括闭源许可证，但对原软件的署名和通知义务继续有效。
- **责任限制：** 按“AS IS”提供，不作保证；在法律允许范围内，原作者和版权持有人不承担相关损失或索赔责任。

**发布前仍需填写：** `LICENSE` 中的 `[YEAR]` 与 `[COPYRIGHT HOLDER / ORGANIZATION]` 仍为占位符；**通知邮箱已设置为 `wuOO364@gmail.com`**。发布到公共仓库后，该邮箱会公开显示在许可证和 README 中。

**当前修订包状态：** 根目录原 AGPL v3 许可证全文已替换为 PANL 1.0，并更新自有浏览器、Rust 和反向分析适配器的许可标记及其锁定文件。仍须由有权许可的版权持有人确认此许可证变更。**第三方依赖的许可证不受该变更影响**：`tools/reverse_mcp/package-lock.json` 中可选上游 `@jshookmcp/jshook` 等依赖仍带有 AGPL 声明，必须分别遵守；Go 辅助组件的 [GO-LICENSE](native/worker/GO-LICENSE) 和 [GO-PATENTS](native/worker/GO-PATENTS) 也必须保留。

PANL 1.0 为自定义许可证名称。若项目工具需要 SPDX 表达式，可考虑以 `LicenseRef-PANL-1.0` **作为自定义引用**并随附许可证全文；不要把它当作已收录的标准 SPDX 标识。
