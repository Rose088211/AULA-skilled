# dsh-TUI · 交互与命令全集

> 入口见 `../SKILL.md`。本文是快捷键、鼠标、问卷与 slash command 的完整参考。

## 快捷键

| 键 | 功能 |
|---|---|
| `Enter` | 发送（`Shift+Enter` 换行）；命令菜单打开时执行选中项 |
| `Ctrl+C` | 中断当前回合；空闲时连按两次退出 |
| `Esc` | 关闭命令/文件菜单；空闲双击清空输入；**空输入双击 = 时间回溯（rewind）** |
| `Ctrl+O` | 展开/收起详情（思考全文、工具参数与输出） |
| `Ctrl+R` | 历史消息搜索 |
| `/` | 会话内全文搜索（`n`/`N` 跳转） |
| `Tab` / `Enter` | 命令 / `@` 文件补全（目录可继续深入） |
| `Ctrl+V` | 粘贴文本或文件管理器中的文件；图片显示为 `[Image #N]` 并作为持久附件发送 |
| `Ctrl+X` | 用 `$VISUAL`/`$EDITOR` 打开当前输入编辑，保存退出后回填 |
| `?` | 快捷键菜单 |
| `Shift+↑` | 消息选择模式（Enter 展开单条） |

**macOS 修饰键**：上表 `Ctrl+<键>` 在 macOS 上同时可用 `⌘<键>`（`⌘V` 粘贴、`⌘O` 展开详情、`⌘Enter` 立即发送）；仅 `Ctrl+C` / `Ctrl+D`（中断/退出）保持 Ctrl 不变，避免与系统级 `⌘C` 复制冲突。`⌘` 需终端支持扩展键盘协议（iTerm2 / kitty / WezTerm / ghostty / tmux）；Terminal.app 会自行消费 `⌘`，请继续用 `Ctrl`。

## 鼠标（`fullscreen: true` 全屏模式；默认关，profile 补丁层覆盖开启）

| 操作 | 功能 |
|---|---|
| 拖拽选择 | 应用内文本选区，**松开即复制**（OSC 52 + `wl-copy`/`xclip`/`xsel` 原生兜底；tmux 内走 `load-buffer -w`），复制后自动取消选区并提示 |
| 双击 / 三击 | 选词 / 选行，即选即复制 |
| 滚轮 | 滚动消息列表 |
| `Esc` | 拖拽进行中取消选区（不复制） |

## 问卷（模型发起 `ask_user_question` 时）

| 键 | 功能 |
|---|---|
| `↑/↓` | 选择选项 |
| `Space` | 多选题勾选/取消 |
| `Tab` | 切到自定义回答（不选选项直接打字） |
| `Enter` | 提交当前选择 |
| `Esc` | 中断提问（模型收到 ASK_ABORTED，可继续对话） |

## 本地命令（`/` 菜单，CC 指令全集复刻，均走 DSH 官方链路）

| 分组 | 命令 |
|---|---|
| 会话 | `/new` 新会话 · `/resume` 会话浏览器（搜索、预览、跨项目、折叠子 agent 运行）· `/rename` 重命名会话 · `/workspace resume\|rename\|open` 管理工作区 · `/clear` 清屏 · `/compact` 压缩 · `/export` 导出 Markdown · `/trace` 轨迹场景（亦可 `Ctrl+T`） |
| 状态 | `/context` 已加载上下文明细 · `/status` 会话信息 · `/cost` token 用量 · `/doctor` 环境自检 · `/config` 配置来源 · `/init` 创建 AGENTS.md |
| 模型 | `/model` 选择器 · `/thinking` 思考显示 · `/tokens` token 明细 · `/theme` 主题选择器 · `/lang` 中英界面切换 |
| 账号/策略 | `/provider` 添加模型提供方 · `/login` 凭证状态 · `/logout` 登出说明 · `/permissions` 权限说明 · `/add-dir` 文件策略范围 · `/hooks` · `/mcp` |
| 技能 | `/audit` 代码审计 · `/bug` bug 报告 · `/review` 代码评审 · `/practice` 编程练习 · `/pr_comments` PR 评论 · `/release-notes` 发布说明 · `/vuln-check` 漏洞检查 |
| 其它 | `/agents` 子代理列表 · `/update` 自动更新并重启 · `/vim` · `/terminal-setup` · `/connect` · `/help` · `/exit` |
| 注册表 | `/plan` `/goal`（DSH 命令注册表插件，随插件自动并入 `/` 菜单） |

## 会话工作流细节

- **时间回溯**：空输入双击 `Esc` → 选一条消息，把对话回退/分叉（rewind/fork）到该处；历史保留，可再 `/resume`。
- **`/model` 切换 = "会话 fork 续聊"**（DSH 无原位换模型 API）：历史原样保留，新会话路由到新模型，旧会话留在 `/resume` 列表；选择写入 `~/.dsh-tui/model.json`，重启与 `/new` 均沿用。
- **`/resume` 会话浏览器**：跨项目搜索、预览、重命名/删除；子 agent 运行可折叠。
- **`/compact`**：压缩上下文后继续，原消息在 `/resume` 里仍可恢复。
- **`/btw` 侧问**：不打断当前回合，向模型追加旁路问题。
- **`/export`**：导出 Markdown；**`/trace`**：轨迹场景复盘。
- 退出时以进程退出收尾，不等待 agent 异步落盘（持久化由 DSH persistence 插件兜底）。
