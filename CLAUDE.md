# AULA-skilled 工作区规则

## 开始工作

- 先读取 `README.md`、`git status --short --branch` 和顶层清单。
- 项目结构不明确时使用 `project-bootstrap`，以实际文件和可验证命令为准。
- 先说明改动范围和验证方式，再开始写入文件。

## 并发协作

- Codex 与 Claude Code 可能同时工作。不要运行 `git reset`、`git clean`、批量删除或覆盖未知来源的修改。
- 编辑前后都检查 Git 状态；只修改当前任务拥有的文件。
- 发现同一文件有其他 Agent 的改动时，先读取并在现有内容上工作，不能静默回滚。

## 技能选择

- React/Next.js 使用 `vercel-react-best-practices`；界面任务结合 `frontend-design` 和 `web-design-guidelines`。
- 浏览器验证使用 `playwright-cli` 或 `webapp-testing`。
- 代码审查使用 `code-review`；提交前用 `git-safe-commit` 做安全审查与分组、`git-commit` 生成规范信息，最后跑 `release-readiness`。
- API 变更先使用 `api-contract`；需要 MCP 时使用 `mcp-builder`。
- 面向中文开发者的文档使用 `chinese-project-docs`，完成后检查 `git diff --check`。

## 安全与验证

- 不把密钥、真实凭据、私有 URL 或本机绝对路径写入仓库。
- 未确认的命令标记为假设，不要伪造测试通过结果。
- 用户未明确要求时不要 commit、push、deploy 或发布外部资源。
