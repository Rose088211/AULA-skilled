# dsh-TUI · 跨 agent 协作协议（已实测）

> 适用场景：另一个 agent（codex / claude / gemini / dsh 等）在 Orca 中启动 dsh-tui、注入任务、读取结果、协作干活。
> 本文所有命令均在本机 **orca 1.4.184 + dsh-tui 0.8.1** 上实测验证，标注 ⚠️ 的是实测踩到的坑。

## 一、前置检查

1. `orca status --json`：确认 Orca runtime ready。
2. `dsh --version`：确认 dsh CLI；`npm ls -g @deepseek-harness-tui/dsh-tui`：确认插件版本。
3. **凭证检查（⚠️ 实测坑 + 根因）**：`orca terminal create` 新起的 PTY **不一定继承发起方 shell 的 env**。实测 `--command "dsh-tui"` 的终端里 `DEEPSEEK_API_KEY` 未配置——模型回合不会执行（界面显示"完工"，token 仍 0）。
   - **根因**：orca 的 PTY env 是 `...process.env` 继承，即 **Electron 应用启动时的 env 快照**。之后新增/修改的系统环境变量（Machine 级）不会出现在任何 orca 终端里——即使 Machine 级已设置（`[Environment]::GetEnvironmentVariable('DEEPSEEK_API_KEY','Machine')` 有值）。dsh 的凭证机制只认环境变量（`apiKeyEnv`），/provider 输入的 key 不落盘，所以每次都要输。
   - **治本**：完全退出 orca 应用再启动（进程树拿到新快照）。验证：新终端 `$env:DEEPSEEK_API_KEY.Length` 有值，或 dsh-tui 里 `/doctor` 显示 `API key: configured`。
   - **不重启 orca 的兜底**：PowerShell profile 自动注入——
     ```powershell
     if (-not $env:DEEPSEEK_API_KEY) {
         $machineKey = [Environment]::GetEnvironmentVariable('DEEPSEEK_API_KEY', 'Machine')
         if ($machineKey) { $env:DEEPSEEK_API_KEY = $machineKey }
     }
     ```
     写入 `~/.dsh-tui` 用户 profile（`$PROFILE`，Windows 为 `Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1`），新终端自动就位（已实测 `AUTO_INJECT=True`）；orca 重启后依然无害。
   - 对策速记：启动后先注入 `/doctor` 检查（输出 `API key: not configured (DEEPSEEK_API_KEY)` 即缺失）。

## 二、启动与等待就绪

```text
ORCA terminal create --worktree active --command "dsh-tui" --json
ORCA terminal wait --terminal <handle> --for tui-idle --timeout-ms 90000 --json
```

- **实测：dsh-tui 虽未被 orca 识别为 agent，`tui-idle` 依然会触发**（通用 idle 兜底）。就绪标志：读输出出现鲸鱼 splash + 输入提示 `❯`。
- 首次启动 profile 会自举安装插件（需 pnpm），等待时间放宽到 90s+。

## 三、注入任务与等待回合

```text
ORCA terminal send --terminal <handle> --text "<任务>" --enter --json
ORCA terminal wait --terminal <handle> --for tui-idle --timeout-ms <长> --json
```

- 任务文本直接进输入框并提交（等价手工输入 + Enter）。send 返回 `accepted: true` 且 `bytesWritten` 正确即成功。
- 回合中追加问题：`/btw <问题>`（侧问，不打断当前回合）。
- 打断回合：发 `Ctrl+C`；空闲时连按两次 `Ctrl+C` 退出整个 TUI。
- 输入残留（⚠️）：若 send 后未提交（如 Enter 时序问题），状态栏右侧会残留任务文本；重发 `/status` 确认输入为空，必要时补一次 Enter 或 `Esc` 清空。

## 四、读取输出（⚠️ 重要细节）

- **全文读（首选）**：`ORCA terminal read --terminal <handle> --json`——返回当前屏幕全文，含面板/回复。
- **增量读**：`--cursor <n>` 只返回**新落的行**；dsh-tui 用原地光标重绘渲染面板与回复，增量读可能看不到内容（实测 `/status` 面板增量读返回 0 行，全文读可见）。
- 判状态：全文读 + 关键词匹配（`完工咯`/`❯`/`Status idle`），或发 `/status` 再全文读。
- **token 计数是"模型是否真跑过"的最可靠信号**：回合后 `/status` 若 `Tokens 0 in → 0 out`，说明模型调用未执行——先查凭证（`/doctor`）。

## 五、会话与收尾

- 会话 id：`/status` 的 `Session <uuid>`；磁盘位置 `~/.dsh/sessions/<编码项目路径>/<uuid>/session.jsonl.zstd`。
- 恢复：`dsh-tui --resume <id>`，或在 orca 里 `terminal create --command "dsh-tui --resume <id>"`。
- 优雅退出：`/exit` + Enter（⚠️ 实测 15s 内未退出，不要死等）。
- 强制收尾：`ORCA terminal close --terminal <handle>`（返回 `ptyKilled: true`；会话由 DSH persistence 兜底，可再 resume）。

## 六、协作模式建议

- **文件系统是主信道，终端是控制信道**：任务/产物通过共享工作区（同一 worktree）传递；dsh-tui 终端只负责下发指令、读状态、接管输出。两侧 agent 改同一份代码即协作。
- 双向协作：A agent 给 dsh-tui 下发子任务并等 `tui-idle` 收结果；dsh-tui 侧需要对外交付时写文件即可，无需 TUI 侧有任何特殊协议。
- 标签页标题（⚠️ 必读）：dsh-tui 终端**必然**被 orca 误标成 "Gemini CLI"——dsh-tui 标题含 `✦`，而 orca 把 `✦` 当 Gemini 专属 glyph（根因与治本补丁见 `orca-integration.md` 的"标签错标"小节）。`terminal rename` 会被 dsh-tui 的下一次标题更新覆盖，**不能靠它治本**；治本需 patch 运行时副本（改 `✦` → `•`，升级后重打）。判断终端里跑的是什么，别依赖标签。

## 七、已验证清单（本机实测）

| 步骤 | 结果 |
|---|---|
| `terminal create --command "dsh-tui"` | ✅ 完整启动（splash + `❯` 输入提示） |
| `terminal wait --for tui-idle`（启动 / 回合结束） | ✅ `satisfied: true` |
| `terminal send` 注入 `/status` `/doctor` | ✅ 命令执行、输出可读 |
| `terminal send` 注入真实任务 | ⚠️ 链路通；模型回合因 PTY 缺 `DEEPSEEK_API_KEY` 未执行（`/doctor` 精确定位） |
| `terminal read`（全文 / 增量） | ✅ 全文可读；增量读对原地重绘内容不可靠 |
| `terminal close` 收尾 | ✅ `ptyKilled: true` |
