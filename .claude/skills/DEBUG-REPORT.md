# AULA-skilled 调试报告

- 执行角色：Hermes（按 Claude Code 任务 `task_ce2f44a17db4`）
- 执行范围：`.claude/skills/` 下全部 35 个技能；未修改技能内容，未提交。
- 执行时间：2026-08-13
- 工作区基线：`git status --short --branch` → `## main...origin/main [ahead 1]`；调试期间未发现并发改动。

## 1. 结论摘要

| 状态 | 数量 |
|---|---:|
| PASS | 29 |
| CONDITIONAL-PASS | 0 |
| NOT-TESTABLE | 6 |
| FAIL | 0 |
| **合计** | **35** |

结构校验未发现不可解析 frontmatter、缺失 `name`/`description`、无法解析的仓库内引用或 Python 语法错误。6 个技能因环境缺少可选运行依赖/目标而无法完成真实产物冒烟，不代表技能内容错误。

## 2. 实际执行证据

### 2.1 结构校验（35/35）

执行了 Python 校验脚本，实际结果：

- 扫描 `.claude/skills/*/SKILL.md`：`count=35`。
- 逐文件解析 YAML frontmatter，并断言 `name`、`description` 为非空字符串：无失败。
- 扫描 Markdown 相对链接及 `references/`、`scripts/`、`assets/`、`agents/` 引用：除示例生成文件 `.playwright-cli/page-...yml` 外无缺失目标；该路径是文档示例快照，不是仓库必需资产。
- 对仓库内 51 个 Python 文件运行 `python -m py_compile`：`python_syntax_failures=[]`。
- `api-contract/references/openapi-template.yaml` 用 PyYAML 解析成功。

### 2.2 集成一致性

执行结果：

- 技能目录：35。
- `skills-lock.json`：26 个外部技能；lock 中技能全部存在。
- 未被 lock 覆盖的 9 个目录正是本地原创技能：`api-contract`、`changelog-generator`、`chinese-humanizer`、`chinese-project-docs`、`cli-wrapper`、`git-commit`、`git-safe-commit`、`project-bootstrap`、`release-readiness`。
- 双目录副本 `.claude/skills` 与 `.agents/skills` 的 4 个重复技能 SHA-256 均一致：`code-review`、`frontend-design`、`playwright-cli`、`vercel-react-best-practices`。
- 执行前后 `git status` 均为 `main...origin/main [ahead 1]`；本任务只新增本报告。

### 2.3 环境能力检查

实际环境检查结果：

- `yaml` 可用。
- `python-docx`、`openpyxl`、`fitz`、`reportlab`、`playwright`、`fastmcp` 均不可用。
- `node`、`npx` 可用；`playwright`、`playwright-cli` 命令不可用。

因此 docx/xlsx/pdf 的最小文件生成、Playwright 页面交互、MCP stub 运行不能在本环境诚实完成，均标记 `NOT-TESTABLE`，没有伪造通过结果。

## 3. 逐技能结果

| 技能 | 状态 | 证据/说明 |
|---|---|---|
| api-contract | PASS | frontmatter/引用通过；OpenAPI 模板 YAML 实际解析成功 |
| brainstorming | PASS | frontmatter、引用与 Python 语法检查通过 |
| changelog-generator | PASS | frontmatter、引用与 Python 语法检查通过 |
| chinese-humanizer | PASS | frontmatter、引用与 Python 语法检查通过 |
| chinese-project-docs | PASS | frontmatter、引用与 Python 语法检查通过 |
| claude-api | PASS | frontmatter、引用与 Python 语法检查通过 |
| cli-wrapper | PASS | frontmatter、引用与 Python 语法检查通过 |
| code-review | PASS | frontmatter、引用与 Python 语法检查通过；副本 SHA-256 一致 |
| dispatching-parallel-agents | PASS | frontmatter、引用与 Python 语法检查通过 |
| docx | NOT-TESTABLE | 需要 `python-docx`/Office 相关能力；当前环境未安装 |
| executing-plans | PASS | frontmatter、引用与 Python 语法检查通过 |
| finishing-a-development-branch | PASS | frontmatter、引用与 Python 语法检查通过 |
| frontend-design | PASS | frontmatter、引用与 Python 语法检查通过；副本 SHA-256 一致 |
| git-commit | PASS | frontmatter、引用与 Python 语法检查通过 |
| git-safe-commit | PASS | frontmatter、引用与 Python 语法检查通过 |
| mcp-builder | NOT-TESTABLE | 文档通过；当前无待运行 MCP stub，且 `fastmcp` 未安装 |
| pdf | NOT-TESTABLE | 需要 PDF 生成/读取依赖；当前环境未安装 |
| playwright-cli | NOT-TESTABLE | 文档结构通过；`playwright-cli` 命令不可用，且示例快照路径不属于仓库资产 |
| project-bootstrap | PASS | frontmatter、引用与 Python 语法检查通过 |
| receiving-code-review | PASS | frontmatter、引用与 Python 语法检查通过 |
| release-readiness | PASS | frontmatter、引用与 Python 语法检查通过 |
| requesting-code-review | PASS | frontmatter、引用与 Python 语法检查通过 |
| skill-creator | PASS | frontmatter、引用与 Python 语法检查通过 |
| subagent-driven-development | PASS | frontmatter、引用与 Python 语法检查通过 |
| systematic-debugging | PASS | frontmatter、引用与 Python 语法检查通过 |
| test-driven-development | PASS | frontmatter、引用与 Python 语法检查通过 |
| using-git-worktrees | PASS | frontmatter、引用与 Python 语法检查通过 |
| using-superpowers | PASS | frontmatter、引用与 Python 语法检查通过 |
| vercel-react-best-practices | PASS | frontmatter、引用与 Python 语法检查通过；副本 SHA-256 一致 |
| verification-before-completion | PASS | frontmatter、引用与 Python 语法检查通过 |
| web-design-guidelines | PASS | frontmatter、引用与 Python 语法检查通过 |
| webapp-testing | NOT-TESTABLE | 需要 Playwright CLI/浏览器；当前环境不可用 |
| writing-plans | PASS | frontmatter、引用与 Python 语法检查通过 |
| writing-skills | PASS | frontmatter、引用与 Python 语法检查通过 |
| xlsx | NOT-TESTABLE | 需要 `openpyxl`/表格运行能力；当前环境未安装 |

## 4. 风险与后续建议

1. **P1：补齐可选依赖后重跑功能冒烟**：在隔离虚拟环境安装 `python-docx`、`openpyxl`、PDF 工具链、`fastmcp` 与 Playwright，再对 6 个 NOT-TESTABLE 技能生成真实 artifact 并验证可读/可运行。
2. **P2：检查 `playwright-cli` 文档示例快照**：`.playwright-cli/page-2026-02-14T19-22-42-679Z.yml` 是示例输出引用；若项目要求“所有引用必须落盘”，应改为明确的示例说明或补充示例资产。当前未改动上游技能文件。
3. **P2：由 Codex 独立复核敏感信息扫描**：本轮只做了结构/引用检查；应按验收计划对全库执行凭据、私有 URL、内网 IP、个人信息扫描，并人工审查脚本安全性。
4. **不建议在本轮修改 lock**：9 个原创技能未进入外部 lock，和仓库 README 的设计一致，不属于漂移。

## 5. 文件变更

仅新增：`.claude/skills/DEBUG-REPORT.md`。

未执行：`git push`、`git commit`、`git reset`、`git clean`、依赖安装、技能内容改写。
