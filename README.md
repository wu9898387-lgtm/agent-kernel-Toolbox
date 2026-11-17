# Agent-Kernel-Toolbox

**减少截图与视觉推理开销，致力于大幅降低 AI 软件及网页交互的 Token 消耗和长期使用成本**

**Agent-Kernel-Toolbox** 以**工具包 + MCP + Skills** 的形式整合浏览器、应用与程序分析能力。通过结构化读取网页 DOM/ARIA、原生控件和程序运行信息，辅助 AI 更高效、深入地理解软件，减少反复截图、视觉识别与试探式操作带来的开销。项目面向能够调用本地工具、控制计算机的 AI 客户端；访问在线网页还需网络连接，**不适用于无法调用外部工具的纯对话模式**

`Python 3.10+` · `MCP stdio` · `Skills` · `PANL 1.0` · `按配置授权工具和目标`

## 当前版本

[下载 Agent-Kernel-Toolbox 0.6.2 Runtime](Agent-Kernel-Toolbox-0.6.2-runtime.zip)

| 项目 | 当前发行包 |
| --- | --- |
| 运行时版本 | `0.6.2` |
| 工具目录 | 223 项工具、55 个工具族 |
| Python 实现 | `runtime-access.pyz`，内含 74 个 Python 源文件 |
| MCP 传输 | 本地子进程 `stdio`，逐行 JSON-RPC 2.0 |
| MCP 协议 | `2025-11-25`、`2025-06-18` |
| 基础环境 | Python 3.10+，原生能力依赖宿主平台和实际配置 |
| 浏览器组件 | Node.js 24+、Playwright 和可运行的浏览器 |

## 快速开始

解压并核对 ZIP 内容及 MCP 连接

```bash
unzip Agent-Kernel-Toolbox-0.6.2-runtime.zip -d Agent-Kernel-Toolbox
cd Agent-Kernel-Toolbox
python3 verify-runtime.py ../Agent-Kernel-Toolbox-0.6.2-runtime.zip
python3 tools/check_installation.py --mcp
```

安装检查覆盖归档、运行时元数据、工具目录和两个协议版本的真实 stdio 调用，覆盖快照、离线回放和纯整数 IR。浏览器、原生进程、远程设备和可选分析后端需按配置分别验证

创建诊断配置和 MCP 客户端条目。将以下路径换成当前用户控制的实际绝对路径，并为本次配置选择独立的状态目录

```bash
python3 tools/configure_runtime.py \
  --config /absolute/private/toolbox-config.json \
  --state-directory /absolute/private/toolbox-state \
  --mcp-config /absolute/private/toolbox-mcp.json \
  --extensions
```

将生成的条目添加到支持本地 stdio MCP 的客户端。运行时按固定配置公开工具、进程、设备和来源。客户端调用时使用 `tools/list` 返回的参数定义

## 0.6.2 变更

- 取消排队的读请求会立即释放队列名额。超长消息在有界读取内找到帧边界时，同一连接可继续处理请求。
- 短窗口和空窗口可沿用同一分块参数。已结束的回放复用原检查点，无需分配新槽位。
- 未注册目标或启动前无法解析的可执行文件会返回明确的未派发结果。可执行文件恢复后可以重试。
- 平台工具可读取完整运行时配置，编译沿用原配置的状态目录和授权。安装命令支持从当前目录解析相对配置路径。
- 浏览器控件和输入框的观察凭证可并存。重叠隐藏在最后一个凭证释放后恢复。快照按响应大小截断，已关闭标签页释放名额。
- 安装检查分别保留两个 MCP 协议的结果。报告写入失败和损坏压缩包均返回明确错误。

发行包 SHA-256

```text
c752d5e7ef3c7dfd5b04cf2f0c72c411dfc4684d5918de04c130ca831b968902
```

## 历史版本 0.5.7.2

- `process.detach` 释放附加会话及其资源，目标进程继续运行
- 新启动的 owned 进程默认使用 `terminate_on_host_exit`，可通过固定 profile 选择 `preserve`
- 多个 Linux Host 可通过同一私有 `coordination_directory` 共享请求去重、来源治理和进程租约，每个 Host 仍使用独立的 `state_directory`
- `runtime.status` 可独立返回最近完成操作的缓存状态、当前调用和忙闲信息。普通调用共用有界串行队列，响应按 JSON-RPC ID 匹配
- 内存捕获同时应用全局和单会话额度。编译及 native lift 缓存增加锁等待上限，并可恢复已完成且通过校验的暂存条目
- 权限拒绝与已派发后的未知效果分别记录。迁移时保留原配置、状态目录和未决记录

## 安全、审计与异常恢复

对可能改变目标的操作，使用独立且不可复用的 `request_id`。JSON-RPC 的 `id` 用来匹配响应。调用方应检查 JSON-RPC `error`、MCP `isError`、`structuredContent.ok` 及返回的效果字段

操作超时或效果为 Unknown 时，保留状态目录并独立核实目标状态。完成核实后，可按发行说明离线记录对账结论

## 包内文档与重建

- `skills/agent-kernel-toolbox/SKILL.md` 描述客户端握手、调用、审计和资源关闭约定
- `runtime-manifest.json` 记录实际版本、能力范围和组件入口
- `tools/extract_sources.py --output src` 将内嵌 Python 源文件提取到尚不存在的目录
- `build-runtime.py` 按清单重建 ZIP 并生成新的 `CONTENTS.json`

```bash
python3 build-runtime.py --output ../Agent-Kernel-Toolbox-0.6.2-rebuilt.zip
python3 verify-runtime.py ../Agent-Kernel-Toolbox-0.6.2-rebuilt.zip
```

## 许可证

仓库提供 [**Permissive Attribution and Notice License (PANL) 1.0**](LICENSE)。完整、具有约束力的英文条款请以同目录 `LICENSE` 为准。以下是便于阅读的中文概要，不替代许可证原文：

- **允许的使用方式：** 免费使用、复制、修改、合并、发布、分发、再许可和销售；允许商业用途和专有/闭源项目，无需公开衍生代码
- **署名与保留：** 在副本和衍生作品中保留原版权声明、作者/组织署名和 PANL 1.0 全文；闭源发行应在随附文档、法律声明或可访问的许可证/关于页面展示署名
- **事先邮件通知：** 在**商业使用**或**纳入专有/闭源项目**之前，向 **[wuOO364@gmail.com](mailto:wuOO364@gmail.com)** 发送邮件，说明 (a) 项目/产品与责任主体、(b) 预期用途（包括商业或闭源用途）、(c) 明确排除的用途。**发送邮件后即可开始使用，无需等待批准、确认或回复。** 已通知的项目后续版本和衍生作品，在预期用途没有实质变化时无需重复通知
- **衍生作品：** 可以采用其他许可证，包括闭源许可证，但对原软件的署名和通知义务继续有效
- **责任限制：** 按“AS IS”提供，不作保证；在法律允许范围内，原作者和版权持有人不承担相关损失或索赔责任

**发布前仍需填写：** `LICENSE` 中的 `[YEAR]` 与 `[COPYRIGHT HOLDER / ORGANIZATION]` 仍为占位符；**通知邮箱已设置为 `wuOO364@gmail.com`**。发布到公共仓库后，该邮箱会公开显示在许可证和 README 中
