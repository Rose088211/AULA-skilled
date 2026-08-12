# AULA-skilled

AULA-skilled 是面向 Claude Code 与兼容 Agent 的技能库，共 **35 个技能**：26 个外部锁定技能，以及 9 个本地原创技能。

## 技能索引

完整说明、来源、许可证与维护命令见 [`.claude/skills/README.md`](.claude/skills/README.md)。

### 外部锁定技能（26）

由 `skills-lock.json` 管理来源与哈希：

`brainstorming` · `claude-api` · `code-review` · `dispatching-parallel-agents` · `docx` · `executing-plans` · `finishing-a-development-branch` · `frontend-design` · `mcp-builder` · `pdf` · `playwright-cli` · `receiving-code-review` · `requesting-code-review` · `skill-creator` · `subagent-driven-development` · `systematic-debugging` · `test-driven-development` · `using-git-worktrees` · `using-superpowers` · `vercel-react-best-practices` · `verification-before-completion` · `web-design-guidelines` · `webapp-testing` · `writing-plans` · `writing-skills` · `xlsx`

### 本地原创技能（9）

不在外部 lock 中，按仓库内容维护：

`api-contract` · `changelog-generator` · `chinese-humanizer` · `chinese-project-docs` · `cli-wrapper` · `git-commit` · `git-safe-commit` · `project-bootstrap` · `release-readiness`

## 使用与维护

Claude Code 会按任务自动加载相关技能；每个技能以 `SKILL.md` 为入口。安装、升级与许可证说明请以 [技能库 README](.claude/skills/README.md) 为准。

本仓库使用 MIT 许可证；部分外部文档技能的来源许可证与使用条件请见技能库 README 中的说明。
