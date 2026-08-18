---
name: dsh-tui
description: 使用 DeepSeek Harness 的官方 TUI 前端 dsh-TUI：安装启动、常用快捷键与命令、会话工作流（/resume /compact /export、rewind/fork）、配置与数据位置、故障排查，以及在 Orca 项目终端中启动它时的集成现状与识别原理。Use whenever the user mentions dsh-tui, dsh TUI, DeepSeek Harness 终端界面, deepseek harness tui, dsh --profile dsh-tui, 在 orca 里跑 dsh-tui, orca 里的 dsh 终端, or asks to launch, operate, resume, update, or debug the dsh-tui terminal interface. 详细键位表、命令全集、配置参考、Orca 识别原理与排查手册见本技能的 references/ 目录。
---

# dsh-TUI 通用指南（入口）

## 这是什么

**dsh-TUI**（npm 包 `@deepseek-harness-tui/dsh-tui`）是 **DeepSeek Harness（DSH）agent 的交互式终端前端**——DeepSeek 生态里的 "Claude Code 终端界面"。它不是独立 agent，而是 DSH 官方 `dsh` CLI 的一个 **profile 插件**（纯插件挂载、零核心改动）：

```text
dsh profile -> dsh-base -> dsh-TUI Cordis patch -> Agent preset + DSH services
  -> session/event 事件流 -> React + 移植 Ink/Yoga 渲染器 -> 终端
```

- **TUI 只负责交互与呈现**；模型调用、工具执行、fork/resume、compaction、持久化由 DSH 服务拥有，会话日志是"对话真源"。
- `dsh-tui` 命令 = 启动器，等价 `dsh --profile dsh-tui`；首次运行自动初始化 profile。
- 仓库：<https://github.com/ccch1mneyyy/dsh-TUI>（MIT）。本机版本以 `npm ls -g @deepseek-harness-tui/dsh-tui` 为准。

## 快速上手

前置：TTY、`dsh` CLI、`pnpm` 10+、Node `^22.19 || >=24`、`DEEPSEEK_API_KEY`。

```sh
npm install -g @deepseek-ai/dsh @deepseek-harness-tui/dsh-tui   # 安装
dsh-tui                                                          # 启动（首次自动初始化 profile）
dsh-tui --resume [id]                                            # 恢复会话（-c / --continue 等价）
dsh --profile dsh-tui                                            # 等价形式
dsh-tui /path/to/project                                         # 打开指定工作区
```

更新：TUI 内 `/update`（自动更新并重启恢复会话），或 `dsh plugin --profile dsh-tui add @deepseek-harness-tui/dsh-tui@latest`。

## 常用操作速查

| 操作 | 做法 |
|---|---|
| 发送 / 换行 | `Enter` / `Shift+Enter` |
| 中断当前回合 | `Ctrl+C`（空闲连按两次退出） |
| **时间回溯（rewind/fork）** | 空输入双击 `Esc` → 选消息回退/分叉 |
| 展开详情（思考、工具参数） | `Ctrl+O` |
| 历史搜索 / 会话内搜索 | `Ctrl+R` / `/`（`n`/`N` 跳转） |
| 命令与 `@` 文件补全 | `Tab` |
| 粘贴文本/图片 | `Ctrl+V`（图片成持久附件 `[Image #N]`） |
| 快捷键菜单 | `?` |

高频命令：`/resume` 会话浏览器（跨项目搜索）· `/new` · `/compact` 压缩 · `/export` 导出 Markdown · `/model` 切换模型 · `/theme` · `/lang` 中英切换 · `/doctor` 环境自检 · `/status` 会话信息 · `/context` 上下文明细 · `/btw` 侧问 · `/update` 更新。macOS 可用 `⌘` 替代多数 `Ctrl` 组合（见 references/interaction.md）。

## 在 Orca 中使用（速览）

dsh-tui 通常在 **Orca 中某项目的终端里手动启动**（cwd 即该项目），Orca 目前把它当作**普通终端**：可 split、读写输出、发键，但**没有** agent 标签身份、状态钩子、prompt 注入——因为 orca 的识别机制（`TUI_AGENT_CONFIG` 清单、进程识别、OSC 标题 token 表）里都没有 dsh-tui。完整机制与接入方向见 `references/orca-integration.md`。

**别的 agent 要调用并协作 dsh-tui**（codex / claude / gemini 等通过 orca 终端驱动）：直接照 `references/cross-agent-collaboration.md` 的协议执行——`terminal create` 启动 → `wait tui-idle` 就绪 → `send` 注入任务 → 再 `wait tui-idle` → `read` 收结果。该协议已实测：`tui-idle` 对 dsh-tui 有效；**最大坑是 orca 新起 PTY 可能缺 `DEEPSEEK_API_KEY`，模型回合会静默不执行**，启动后先 `/doctor` 验证。标签必被误标 "Gemini CLI"（dsh-tui 标题的 `✦` 与 orca 的 Gemini 检测冲突，rename 会被覆盖）——治本补丁见 `references/orca-integration.md`。

## 故障速查

| 现象 | 处理 |
|---|---|
| 未检测到 dsh CLI / 需 pnpm | `npm install -g @deepseek-ai/dsh` / `npm install -g pnpm`（或 corepack） |
| 报 `ERR_PNPM_ADDING_TO_ROOT` | `dsh plugin --profile dsh-tui add -w @deepseek-harness-tui/dsh-tui@<版本>` |
| 启动器拒绝启动（profile 版本更旧） | 对齐：`dsh plugin --profile dsh-tui add @deepseek-harness-tui/dsh-tui@<启动器版本>` |
| 启动即崩溃 | 直接跑 `dsh --profile dsh-tui` 看完整报错；`dsh --profile dsh-tui --dump-config` 查组合 |

完整排查表见 `references/troubleshooting.md`。

## 参考文档（references/）

| 文件 | 内容 |
|---|---|
| `references/interaction.md` | 完整键位表（含 macOS/鼠标/问卷）、命令全集、会话工作流细节 |
| `references/configuration.md` | 数据路径、环境变量、Agent preset、随包技能、MCP、主题、权限与安全边界 |
| `references/orca-integration.md` | Orca 的 agent 识别机制（源码级）、在 Orca 中操作 dsh-tui、让 Orca 识别它的接入方向 |
| `references/cross-agent-collaboration.md` | **跨 agent 协作协议（已实测）**：另一个 agent 在 Orca 里启动/注入/读取/收尾 dsh-tui 的完整步骤与踩坑清单 |
| `references/troubleshooting.md` | 诊断顺序、版本错位规则、更新路径、已知限制与边界 |

> ⚠️ 安全提示：dsh-TUI 无独立沙箱，沿用 profile 策略；**Windows 无沙箱后端会退到 `danger-full-access` 且不弹审批**。在敏感/不可信仓库中启动前先检查 profile 配置。
