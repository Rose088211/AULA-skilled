# dsh-TUI · 在 Orca 中的集成

> 入口见 `../SKILL.md`。本文解释 Orca 如何识别 agent、为什么认不出 dsh-tui、在 Orca 里怎么操作它，以及未来的接入方向。

## 现状

dsh-tui 通常在 **Orca 中某项目的终端里手动启动**：`orca terminal create --worktree active --command "dsh-tui"`，或直接在项目终端里敲 `dsh-tui`，cwd 即该项目。

**Orca 目前把 dsh-tui 当作普通终端**——可正常 split、读写输出、发键、追踪 cwd，但以下 agent 专属能力不可用：

- agent 标签页身份/图标（tab 不会被标记为某 agent）
- 工作状态钩子（status hooks、完成通知）
- 启动/恢复按钮、prompt 注入（draft prefill）
- 粘贴就绪信号（draft paste ready）

## Orca 的 agent 识别机制（源码级）

orca 源码位置（解包 app.asar 后）：`out/shared/tui-agent-config.js`、`agent-process-recognition.js`、`agent-name-token-match.js`、`agent-kind.js`。

### 1. 可启动 agent 清单：`TUI_AGENT_CONFIG`

约 40 个条目（claude、claude-agent-teams、openclaude、codex、autohand、opencode、pi、gemini、antigravity、aider、goose、cursor、kimi、qwen-code、hermes、copilot、grok、devin、trae…），每条含：

- `detectCmd` / `detectCmdAliases`：检测命令（PATH 上是否存在）
- `launchCmd`：启动命令（可带参数，如 `hermes --tui`、`kiro-cli chat --tui`）
- `expectedProcess`：期望的前台进程名（用于进程识别）
- `promptInjectionMode`：prompt 注入方式——`argv`（如 codex/claude）、`stdin-after-start`（启动后写 stdin）、`flag-prompt`、`flag-prompt-interactive` 等
- 可选：`draftPromptFlag`（如 claude 的 `--prefill`）、`draftPasteReadySignal`、`preflightTrust`、`windowsShiftEnterEncoding`

### 2. 进程识别：`agent-process-recognition.js`

- 前台进程名直接映射（由 `TUI_AGENT_CONFIG` 构建 `PROCESS_TO_AGENT` 表）
- node/python 解释器场景：识别入口点脚本/包路径（如 `node_modules/@openai/codex/`、`node_modules/@google/gemini-cli/`）
- dsh-tui 的前台进程是 `node …/dsh-tui.js` 或 `dsh`，两者都不在识别表；`dsh` 也不是任何 agent 的 launchCmd

### 3. OSC 终端标题 token 匹配：`agent-name-token-match.js`

- `AGENT_NAMES`：claude、openclaude、codex、copilot、cursor、gemini、antigravity、opencode、mimo、openclaw、aider、grok、devin
- 按**整词**匹配（`(?<![\w./\\-])name(?![\w./\\-])`），避免目录名误报
- dsh-tui 的标题形如 `⠂ 🐋 <会话标题>`（Windows 走 `process.title`，其它平台 OSC 0），不含任何已知 agent token → 标题检测也不命中

## 标签错标 "Gemini CLI"：根因与治本（已实测）

**根因（源码级铁证 + 实测复现）**：dsh-tui 的终端标题固定为 `✦ 🐋 <会话标题>`（空闲前缀 `✦`、工作态 `⠂/⠐` 帧，见 `screens/Chat.js`），而 orca 的标题识别（`terminal-title-agent-type.js`）把 `✦`（U+2726）定义为 **Gemini 专属工作 glyph**（`GEMINI_WORKING = '✦'`），`isGeminiTerminalTitle()` 见 `✦` 即判 "Gemini CLI"。**任何 dsh-tui 终端都会被误标**，与创建方式（手动/CLI/quick command）无关。

**为什么 rename 不可靠（实测推翻初版结论）**：`orca terminal rename` 写 customTitle，看似最高优先级，但 orca 的 agent 识别在**每次 dsh-tui 更新标题**（工作/空闲切换、会话标题变化）时重新判定并覆盖——实测 rename 成 "dsh-tui" 后发 `/status` 触发标题更新，标签立刻变回 "Gemini CLI"。**事后改名全部无效，不要用 rename 治本。**

**治本：patch dsh-tui 运行时副本，去掉标题里的 `✦`**（已实测生效，标签变为 `• 🐋`）：

```text
文件：~/.dsh/profiles/dsh-tui/node_modules/@deepseek-harness-tui/dsh-tui/lib/types/screens/Chat.js
原：  const titlePrefix = channel.working ? (TITLE_ANIMATION_FRAMES[titleFrame] ?? '✦') : '✦';
改：  const titlePrefix = channel.working ? (TITLE_ANIMATION_FRAMES[titleFrame] ?? '•') : '•';
```

- 改前先备份（`Copy-Item Chat.js Chat.js.bak-ocrafix`）；改后**重启 dsh-tui 生效**（运行中的实例不重载）。
- **升级会覆盖补丁**：每次 `/update` 或 `dsh plugin --profile dsh-tui add …` 后需要重新 patch（skill 已把此作为维护事项）。
- 兜底：也可顺带执行 `orca terminal rename --title "dsh-tui"` 让标签更明确（补丁后 rename 不再被 ✦ 覆盖，因为标题已无 ✦；但注意会话标题变化不触发 agent 重判）。

**上游修复建议（记录在案）**：
- dsh-tui 上游（ccch1mneyyy/dsh-TUI）：`✦` 是 Claude Code 家族的空闲标题前缀，被 orca 硬编码为 Gemini glyph——建议把空闲前缀改为 `•` 或直接 `🐋`。
- orca 上游（stablyai/orca）：`isGeminiTerminalTitle` 应要求 `✦` 之外同时命中 `gemini` token（或排除 `🐋` 等已知非 Gemini 特征），避免误判 Claude Code 系 TUI。

## 在 Orca 里操作 dsh-tui（供其它 agent）

> 完整可执行协议（启动 → 就绪 → 注入 → 收结果 → 收尾，含实测踩坑）见 `cross-agent-collaboration.md`。以下是速记。

- 用 `orca terminal` 命令读写：读输出判断状态（等待 spinner / 输入提示 / 完成），`terminal send` 发键
- 常用键序列：`Enter` 发送、`Esc` `Esc` 空输入回退、`/resume` + `Enter` 恢复会话、`/status` + `Enter` 看会话信息、`/doctor` + `Enter` 环境自检
- 会话恢复不依赖 Orca：`dsh-tui --resume` 或 TUI 内 `/resume` 浏览器跨项目搜索
- 一次性非交互任务不要开 TUI：`dsh --profile headless "<任务>"`（官方无头模式，跑完打印结果退出）

## 让 Orca 识别 dsh-tui 的接入方向（需 orca 侧改动）

1. **`TUI_AGENT_CONFIG` 增加 `dsh-tui` 条目**：
   - `detectCmd: 'dsh-tui'`、`launchCmd: 'dsh-tui'`
   - `promptInjectionMode: 'stdin-after-start'`（dsh-tui 无 `--prefill` 类 flag，需启动后注入）
   - `expectedProcess` 有难度：实际前台进程是 `node`/`dsh`，需要配合 node 入口点识别（如 `node_modules/@deepseek-harness-tui/dsh-tui/bin/dsh-tui.js`）
2. **`AGENT_NAMES`（标题 token 表）增加 `dsh` / `dsh-tui`**，并让 TUI 标题包含该 token（当前是 `🐋` + 会话标题；可推动 dsh-tui 上游在标题里加 `dsh-tui` 词）
3. 配套：`agent-kind.js` 的 telemetry 映射、launch 面板的 agent 图标/文案

> 社区侧（dsh-tui 上游）能做的：标题加 agent 名 token、暴露稳定的 ready 信号。两者配合才能完整接入。
