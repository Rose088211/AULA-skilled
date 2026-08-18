# AULA-skilled · 技能库

本目录是 `AULA-skilled` 项目的 AI Agent 技能库，供 Claude Code（及兼容 agent）加载。共 **36 个技能**：26 个来自社区/官方，10 个为本地原创。

## 目录

- [原创技能（本仓库）](#原创技能本仓库)
- [官方（anthropics/skills）](#官方-anthropicsskills)
- [软件开发方法论（obra/superpowers）](#软件开发方法论-obrasuperpowers)
- [前端（vercel-labs + 依赖）](#前端-vercel-labs--依赖)
- [自动安装的配套依赖](#自动安装的配套依赖)
- [升级与维护](#升级与维护)

## 原创技能（本仓库）

| 技能 | 一句话用途 |
|---|---|
| `chinese-humanizer` | 中文去 AI 味：把机械、模板化的 AI 中文改写成自然地道、有语感的表达 |
| `changelog-generator` | 从 git 历史生成/更新 CHANGELOG.md（Keep a Changelog + 语义化版本） |
| `git-safe-commit` | 安全的提交流程：审 diff、扫密钥敏感信息、逻辑分组、规范 commit message |
| `api-contract` | REST API 契约设计 + OpenAPI 3 规范生成 |
| `cli-wrapper` | 把函数/脚本封装成可安装、可发布的命令行工具 |
| `dsh-tui` | DeepSeek Harness 的 Claude Code 风格终端界面（dsh CLI 的 TUI profile）：安装启动、会话工作流、配置排查与 Orca 集成 |
| `chinese-project-docs` | 依据仓库事实撰写/维护中文项目文档（README、开发指南、API 说明） |
| `project-bootstrap` | 进入陌生仓库时快速摸清技术栈、包管理、入口命令与风险，建立可执行的工作流 |
| `release-readiness` | 提交/合并/发布前的就绪检查：scope、测试、构建、密钥扫描、逐项给结论 |
| `git-commit` | 依据 diff 自动生成 conventional commit message（类型/scope/描述/分组） |

其中 `chinese-humanizer`/`changelog-generator`/`git-safe-commit`/`api-contract`/`cli-wrapper`/`dsh-tui` 为本仓库手写原创；`chinese-project-docs`/`project-bootstrap`/`release-readiness`/`git-commit` 由环境脚手架自动生成后补全（Source: local）。均可直接复制到其他项目使用。

> `dsh-tui` 同时镜像到共享目录 `.agents/skills/dsh-tui/`（两处内容一致），供 codex / gemini 等按 agents.md 规范读技能的 agent 加载——便于它们在 Orca 里按 `references/cross-agent-collaboration.md` 的协议调用并协作 dsh-tui。

> `git-safe-commit` 与 `git-commit` 分工：前者做提交前的安全审查（diff 审阅、密钥扫描、分组），后者负责生成规范化的 commit message。提交流程建议 `git-safe-commit` 打底、`git-commit` 出信息。

## 官方（anthropics/skills）

许可证：除标注外均为 Apache-2.0；`docx`/`xlsx`/`pdf` 为 **source-available（专有）**，可免费使用，须保留署名与许可声明。

| 技能 | 一句话用途 |
|---|---|
| `skill-creator` | 造新技能/优化已有技能的元技能（含评估循环） |
| `claude-api` | Claude / Anthropic SDK API 参考 |
| `mcp-builder` | 构建 MCP server |
| `webapp-testing` | 用 Playwright 测试本地 Web 应用 |
| `frontend-design` | 有主见、不模板化的前端视觉设计 |
| `docx` | 生成/编辑 Word 文档 |
| `xlsx` | 生成/编辑 Excel 表格 |
| `pdf` | 生成 PDF |

## 软件开发方法论（obra/superpowers）

许可证：MIT。最热门的 agentic 开发方法论框架，技能间互相引用，建议整体使用。

`brainstorming` · `writing-plans` · `executing-plans` · `verification-before-completion` · `systematic-debugging` · `test-driven-development` · `requesting-code-review` · `receiving-code-review` · `subagent-driven-development` · `dispatching-parallel-agents` · `using-git-worktrees` · `finishing-a-development-branch` · `writing-skills` · `using-superpowers`

## 前端（vercel-labs + 依赖）

| 技能 | 来源 | 一句话用途 |
|---|---|---|
| `web-design-guidelines` | vercel-labs/agent-skills | 按 Web 界面规范审查 UI |
| `vercel-react-best-practices` | vercel-labs/agent-skills | React/Next.js 性能优化指南 |

## 自动安装的配套依赖

`npx skills add` 在安装上述技能时自动解析并安装了它们的配套依赖，放进了 `.claude/skills/` 以保证 Claude Code 能读到：

| 技能 | 来源 | 服务对象 |
|---|---|---|
| `playwright-cli` | microsoft/playwright-cli | `webapp-testing` 的浏览器自动化 |
| `code-review` | mattpocock/skills | `web-design-guidelines` 的并行双轴审查 |

> 说明：CLI 会把依赖安装到共享目录 `.agents/skills/`；Claude Code 默认只读 `.claude/skills/`，故本仓库把依赖复制到了 `.claude/skills/`，两处并存、内容一致。

## 升级与维护

技能通过 `vercel-labs/skills` CLI 管理，锁文件 `skills-lock.json` 记录来源与哈希。

```bash
# 列出已装技能
npx skills list

# 升级所有外部技能
npx skills update -p -y

# 新增一个技能（示例）
npx skills add vercel-labs/agent-skills --agent claude-code --copy --skill react-best-practices
```

改动外部技能前请核对各来源仓库的许可证（尤其 anthropics 的文档类技能为 source-available）。

## 使用

Claude Code 会在相关任务中按需加载这些技能，也可以 `/skill <名称>` 直接调用。每个技能都以 `SKILL.md` 开头，含 YAML frontmatter（`name` + `description`）。
