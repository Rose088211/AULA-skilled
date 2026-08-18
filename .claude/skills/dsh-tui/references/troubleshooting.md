# dsh-TUI · 故障排查与维护

> 入口见 `../SKILL.md`。本文是诊断顺序、版本错位规则、更新路径与已知边界的完整手册。

## 诊断顺序

1. `/doctor` — 环境自检（CLI、pnpm、profile、凭证）
2. `/config` — 配置来源（哪些行从哪层来）
3. `/context` — 已加载上下文明细
4. `/status` — 会话信息
5. `/cost` / `/tokens` — 用量明细
6. 启动失败时：直接跑 `dsh --profile dsh-tui` 拿完整诊断输出；`dsh --profile dsh-tui --dump-config` 查看组合后的 profile 树

## 版本错位规则（启动器 vs profile 内安装的版本）

`dsh-tui` 启动器与 `~/.dsh/profiles/dsh-tui` 里安装的插件包版本可能不同步：

- **反向错位（profile 比启动器旧，major/minor 更旧）→ 拒绝启动**。原因：启动器的 bundle patch 会套用到 profile 的旧包上，而旧包缺少 patch 引用的子路径导出，启动必然以模块解析错误崩溃（`ERR_PACKAGE_PATH_NOT_EXPORTED`）。
  - 修复：`dsh plugin --profile dsh-tui add @deepseek-harness-tui/dsh-tui@<启动器版本>`（或 `@latest` 全部升到最新）
- **前向错位（profile 比启动器新）→ 打印一行提示后继续启动**（0.7.2 起有本地 workspace 兜底）；之后在 TUI 内 `/update` 或重新 add 对齐。

## 更新

- TUI 内 `/update`：自动更新并重启恢复当前会话（后台检查 npm 新版本）
- 手动：`dsh plugin --profile dsh-tui add @deepseek-harness-tui/dsh-tui@latest`

## 常见问题表

| 现象 | 处理 |
|---|---|
| `[dsh-tui] 未检测到 dsh CLI` | `npm install -g @deepseek-ai/dsh` |
| `[dsh-tui] 首次安装需要 pnpm` | `npm install -g pnpm` 或 `corepack enable pnpm` |
| 安装报 `ERR_PNPM_ADDING_TO_ROOT` | 带 `-w` 重试：`dsh plugin --profile dsh-tui add -w @deepseek-harness-tui/dsh-tui@<版本>`（pnpm ≥11 在带 pnpm-workspace.yaml 的 profile 目录的已知行为） |
| `无法启动：profile 内运行的是 vX 而启动器是 vY` | 反向错位，按上节对齐命令修复 |
| 启动器提示 profile 版本较新 | 可继续使用；之后在 TUI 内 `/update` 对齐 |
| 启动即崩溃 / 模块解析错误 | 直接跑 `dsh --profile dsh-tui` 看完整报错；再 `--dump-config` 检查组合 |
| `dsh plugin` 报 `required option '--profile <name>' not specified` | 所有 `dsh plugin` 子命令都必须带 `--profile`（如 `dsh plugin --profile dsh-tui list`） |
| 环境变量旧名警告（`CC_TUI_*`） | 按提示改用新名（`DSH_TUI_*`），旧名不再生效 |
| 想换语言 | `/lang` 中英切换，或 `DSH_TUI_LANG=en` |

## 已知限制与边界

- TUI 是**交互式**前门；脚本化/管道/一次性任务用 `dsh --profile headless "<任务>"`，不要试图用 TUI 跑非交互流程。
- `/vim`、`/connect`、`/hooks` 为 Claude Code 同名占位：DSH 侧无等价机制时命令给出明确说明而非静默。
- `/model` 实时切换走"会话 fork 续聊"（无原位换模型 API），旧会话留在 `/resume` 列表。
- `Ctrl+V` 读剪贴板依赖平台工具：Windows 用 PowerShell `Get-Clipboard`（短暂锁定时自动重试），macOS 用 `osascript`/`pbpaste`，Linux 需 `wl-paste`/`xclip`/`xsel` 之一；工具缺失时提示无可用剪贴板工具，不受支持的图片格式走临时文件引用降级。
- 退出时以进程退出收尾，不等待 agent 异步落盘（持久化由 persistence 插件兜底）。
- 注入上下文（plugin source 内容）未做独立展示，随系统提示词并入进度条统计。
- 长会话性能由事件驱动投影、差分输出、消息虚拟化与有界缓存保障，不需要频繁 `/compact`。
- 故障排查以实际命令输出为准；版本信息查 `npm ls -g`、`dsh --version`，不要臆测。
