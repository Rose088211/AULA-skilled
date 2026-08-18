# dsh-TUI · 配置、数据与扩展

> 入口见 `../SKILL.md`。本文是数据路径、环境变量、preset、技能、MCP、主题与安全边界的完整参考。

## 数据与配置位置

| 路径 | 内容 |
|---|---|
| `~/.dsh/profiles/dsh-tui/` | profile 目录：插件 node_modules、cordis 组合（`dsh plugin --profile dsh-tui list` 查看） |
| `~/.dsh-tui/` | TUI 状态：`resume.txt`（上次会话）、`history.jsonl`、`session-index.json`、`effect-ledger.jsonl`、`last-used.json`、`trajectory.json` |
| `~/.dsh-tui/agent-preset.json` | 当前 Agent preset（`/preset` 切换后持久化） |
| `~/.dsh-tui/model.json` | `/model` 选择持久化 |
| `~/.dsh-tui/themes/<名字>.json` | 自定义主题 |
| `~/.dsh/sessions/<编码后的项目路径>/` | DSH 侧会话日志（每个项目一个目录，例如 `--D-orca_project-AULA-skilled--`） |
| `~/.dsh/storages/` · `~/.dsh/.agent-presets/` | DSH 存储与用户 preset 根 |

## 环境变量

| 变量 | 作用 |
|---|---|
| `DEEPSEEK_API_KEY` | 模型凭证（必填才能运行模型） |
| `DEEPSEEK_BASE_URL` | 模型 base URL（可选） |
| `DSH_TUI_LANG` | 界面语言 zh/en，默认 zh；旧名 `CC_TUI_LANG` 不再生效 |
| `DSH_TUI_PERSONA` | persona 覆盖（默认 `You are a coding agent.`） |
| `DSH_TUI_THEME` | 主题（优先级高于持久化选择） |
| `DSH_TUI_RESUME_SESSION` | 启动器内部使用（`--resume` 的会话 id） |
| `DSH_TUI_WORKSPACE_TARGET` | 启动器内部使用（工作区目标：路径或 URL） |
| `DSH_HOME` | dsh 数据根，默认 `~/.dsh` |

启动器还会对旧名环境变量（`CC_TUI_*` 等）打印改名警告，旧名不再生效。

## Agent preset

四种官方模式 + 一个随包模式：

- `standard` / `code` / `minimal` / `cordis`（官方）· `liangshen`（随包"梁神模式"）
- `/preset` 切换：**已产生对话的会话不可切换，空白会话立即生效**
- 默认 preset 持久化在 `~/.dsh-tui/agent-preset.json`

## 随包技能

`/skills` 查看注册表，picker 或 `/命令` 直接使用：

`audit`（代码审计）· `bug`（bug 报告）· `review`（代码评审）· `practice`（编程练习）· `pr_comments`（PR 评论）· `release-notes`（发布说明）· `vuln-check`（漏洞检查）

## MCP

- 通过 `@deepseek-ai/dsh-mcp-client` 挂载服务器，工具以 `mcp__<服务器>__<工具>` 注册
- `/mcp` 查看连接状态

## 主题

- `/theme` 选择器：`auto`（跟随系统/终端背景）+ 内置 `light` / `dark` / `dark-ansi`
- 自定义：`~/.dsh-tui/themes/<名字>.json`，选中即热切换并持久化
- 优先级：`DSH_TUI_THEME` 环境变量 > 持久化选择 > OSC 11 终端背景自动检测
- 配色理念：Gentle Mist Blue——雾蓝只承担品牌/焦点/交互/高亮，正文保持中性灰

## 权限与安全边界

- dsh-TUI **不实现独立沙箱**，沿用当前 DSH profile 的文件、Shell、sandbox 与 approval 策略
- **Windows 当前没有对应沙箱后端：组合退回到 `danger-full-access` 且不弹审批**
- 在包含敏感凭证或不可信仓库的环境中启动前，先检查 profile 配置；查看组合结果：`dsh --profile dsh-tui --dump-config`
- 工具级审批已实现：approval 服务 + TUI answerer（CC 式审批面板）消费审批流；`/permission` 预设切换由 dsh-base 的 `permission-presets` 插件提供

## 生态与扩展

- 插件开发指南见官方仓库 `docs/plugins.md`（会话事件 / 槽位 / 技能 / 主题 / prompt 段接缝）
- 生态组织：[dsh-tui-ecosystem](https://github.com/dsh-tui-ecosystem)，模板仓库 `plugin-template`
- 参考实现：`dsh-working-activity`（实时工作状态行）
- ecosystem-spec：随包携带 `ecosystem-spec/registry` 与 `schemas`，含 host-descriptor（`hostId: dsh-tui`）等 dsh 生态互操作规范
