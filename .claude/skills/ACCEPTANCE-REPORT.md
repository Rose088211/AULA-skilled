# AULA-skilled 验收报告

- 验收角色：Codex（按 Claude Code 任务 `task_601383704828`）
- 验收日期：2026-08-13
- 验收范围：`.claude/skills/` 下 35 个技能及根目录索引、锁文件
- 结论：**ACCEPT-WITH-CONDITIONS**
- 复验日期：2026-08-13；五项内容修复已逐文件核实通过。

## 1. 总结

| 结果 | 数量 |
|---|---:|
| PASS | 29 |
| CONDITIONAL-PASS | 6 |
| FAIL | 0 |
| 合计 | 35 |

Hermes 调试报告的历史摘要曾与逐技能表不一致；Hermes 后续已将调试摘要修正为 `29 PASS / 6 NOT-TESTABLE / 0 CONDITIONAL-PASS`。本报告按当前文件和独立复验结果计数。

## 2. 验收证据

### 2.1 完整性与结构

- `.claude/skills` 实际目录数：35；每个目录均有 `SKILL.md`。
- 独立 YAML frontmatter 扫描：35/35 的 `name`、`description` 均为非空字符串；无重复 skill name；name 与目录名一致。
- Python AST 扫描：51 个 Python 文件，0 个语法错误。
- `.agents/skills` 的 4 个重复技能与 `.claude/skills` 的 `SKILL.md` SHA-256 一致：`code-review`、`frontend-design`、`playwright-cli`、`vercel-react-best-practices`。
- `skills-lock.json` 包含 26 个外部技能，均能对应实际目录；未锁定的 9 个目录均为仓库声明的本地原创技能。

### 2.2 引用与索引

五项条件修复已通过当前文件实证核验：

- 根目录 `README.md` 列出全部 35 个实际技能，并区分 26 个外部锁定技能与 9 个本地原创技能。
- `claude-api/SKILL.md` frontmatter description 为 688 字符，低于 1024 字符限制。
- `vercel-react-best-practices/AGENTS.md` 的三个规则链接均包含 `rules/` 前缀。
- `writing-skills/anthropic-best-practices.md` 的外部文档链接已改为说明性外部引用，不再作为仓库内相对文件解析。
- `playwright-cli/SKILL.md` 已将时间戳快照标注为说明性示例而非仓库资产；独立本地链接扫描为 0 个缺失目标。

### 2.3 安全与脚本审查

独立扫描范围为 `.claude/skills` 全部文本和脚本文件：

- 高置信度真实密钥模式：0。
- 私有域名、邮箱形式的个人真实信息、Windows 私有绝对路径：0 个可确认泄露。
- `localhost`、`127.0.0.1` 命中均为本地开发服务器/浏览器测试示例；没有发现内网地址泄露。`10/8`、`172.16/12`、`192.168/16` 未发现命中。
- `ghp_your_github_token`、`your-api-key` 等均为明确占位符示例，不是有效凭据；但建议统一改成更不易被扫描器误判的 `<GITHUB_TOKEN>` / 环境变量示例。
- 脚本静态审查发现的 `subprocess`、`child_process`、socket、临时文件删除、`soffice` 调用、Playwright 启动等均与各技能的预期功能相符；未发现隐藏下载、凭据外传、持久化、任意路径清理或混淆载荷。
- `brainstorming` 的服务脚本默认绑定 loopback，并有 token/origin 检查；停止脚本删除的是其自身 session 目录，未见越界目标。
- `docx`、`pdf`、`xlsx` 的 `LICENSE.txt` 均随技能保留；Apache 许可技能也保留了对应许可证文件。

### 2.4 功能可用性

Hermes 的结构检查、OpenAPI YAML 解析和 Python 语法检查证据可信；但当前环境缺少 `python-docx`、`openpyxl`、PDF 依赖、`fastmcp` 和 `playwright-cli`，因此以下技能没有完成真实 artifact/浏览器冒烟：`docx`、`mcp-builder`、`pdf`、`playwright-cli`、`webapp-testing`、`xlsx`。

这 6 项不能判为 FAIL，但在补齐隔离测试依赖并完成最小产物可读性/可运行性验证前，只能是条件通过。五项内容修复本身已关闭，不再产生额外条件项。

## 3. 逐技能结论

| 技能 | 结论 | 说明 |
|---|---|---|
| api-contract | PASS | frontmatter、引用、OpenAPI 模板 YAML 解析通过 |
| brainstorming | PASS | 结构、脚本引用、Python/JS 内容检查通过 |
| changelog-generator | PASS | 结构与本地引用通过 |
| chinese-humanizer | PASS | 结构与本地引用通过 |
| chinese-project-docs | PASS | 结构与 agent 配置引用通过 |
| claude-api | PASS | description 复验为 688 字符；正文引用目标存在 |
| cli-wrapper | PASS | 结构与本地引用通过 |
| code-review | PASS | 结构及副本一致；外部 issue-tracker 依赖已按说明性外部资源处理 |
| dispatching-parallel-agents | PASS | 结构与引用通过；测试文件路径属于任务示例文本，不是 Markdown 链接 |
| docx | CONDITIONAL-PASS | 结构通过；未安装 `python-docx`/Office，未完成真实文档产物测试 |
| executing-plans | PASS | 结构与引用通过 |
| finishing-a-development-branch | PASS | 结构与引用通过 |
| frontend-design | PASS | 结构通过；与 `.agents` 副本一致 |
| git-commit | PASS | 结构通过；与安全提交技能职责可区分 |
| git-safe-commit | PASS | 结构、敏感信息规则和职责边界通过 |
| mcp-builder | CONDITIONAL-PASS | 结构通过；未安装 `fastmcp`，未运行 MCP stub |
| pdf | CONDITIONAL-PASS | 结构通过；缺少 PDF 生成/读取依赖，未完成真实 artifact 测试 |
| playwright-cli | CONDITIONAL-PASS | 结构通过但命令不可用；另有缺失示例快照引用 |
| project-bootstrap | PASS | 结构与 agent 配置引用通过 |
| receiving-code-review | PASS | 结构与引用通过 |
| release-readiness | PASS | 结构与 agent 配置引用通过 |
| requesting-code-review | PASS | 结构与引用通过 |
| skill-creator | PASS | 结构、脚本语法通过；校验器自身默认编码不适配当前 Windows 终端，但不是技能正文缺陷 |
| subagent-driven-development | PASS | `todo per task` 是 Graphviz 示例语句，不是未完成 TODO 占位 |
| systematic-debugging | PASS | 结构与脚本引用通过 |
| test-driven-development | PASS | 结构与引用通过 |
| using-git-worktrees | PASS | 结构与引用通过 |
| using-superpowers | PASS | 结构与引用通过 |
| vercel-react-best-practices | PASS | 副本一致；三个规则链接均已补 `rules/` 前缀 |
| verification-before-completion | PASS | 结构与引用通过 |
| web-design-guidelines | PASS | 结构与引用通过 |
| webapp-testing | CONDITIONAL-PASS | 结构通过；缺少 Playwright CLI/浏览器，未完成页面交互测试 |
| writing-plans | PASS | 结构与引用通过；占位词出现在自检规则示例中 |
| writing-skills | PASS | 外部文档引用已明确标注为外部资源，不再形成缺失本地链接 |
| xlsx | CONDITIONAL-PASS | 结构通过；未安装 `openpyxl`/Office，未完成真实表格产物测试 |

## 4. 必须条件与后续动作

1. 在隔离环境补齐文档、PDF、MCP、Playwright 依赖，分别生成最小 docx/xlsx/pdf、运行 MCP stub、完成页面交互，再更新本报告中的 6 个条件项。
2. 对示例凭据持续使用 `<...>` 或环境变量表达，降低误报风险；不应把任何真实 token 写入示例。

在上述条件完成前，不建议将本次验收升级为无条件 `ACCEPT`。
