---
name: git-safe-commit
description: 安全的 git 提交流程——提交前审 diff、扫敏感信息、按逻辑分组、写规范 commit message。Use whenever the user says commit, 提交, 提交代码, stage, git 提交, 帮我 commit, or is about to commit — even if they didn't ask for a review. NEVER run a commit without first checking for secrets and confirming what gets staged.
---

# 安全提交（Git Safe Commit）

提交是发布代码的动作，出问题就进了历史、推上远程就很难擦干净。这个技能让每次提交前都过一遍「审 diff → 扫敏感信息 → 分组 → 规范信息」的关卡。

## 何时使用

任何提交场景：用户说「提交」「commit」「stage 一下」「把改动提交了」，或完成一段工作后自然要提交。如果用户已经说得很明确（如「就按老规矩提交」），仍然要跑敏感信息检查这一步——这是底线，不能省略。

## 流程

### 1. 看清状态和差异

```
git status --short
git diff               # 未暂存
git diff --cached      # 已暂存
git diff HEAD          # 全部改动
```

先搞清楚改了哪些文件、新增了哪些文件。注意 untracked 文件——它们不会被 `git diff` 显示，但要逐个看一遍再决定加不加。

### 2. 敏感信息扫描（不可跳过）

对**所有**改动内容（含新增文件）扫描这些模式，详见 `references/secret-patterns.md`：

- 密钥/令牌：`AKIA`、`sk-`、`ghp_`、`glpat-`、JWT、`BEGIN RSA/OPENSSH PRIVATE KEY`
- 凭据：`password=`、`passwd`、连接串里的 `user:pass@`、`.env` 中的真值
- 地址：内网 IP、带端口的数据库地址、云服务真实地址
- 个人信息：身份证号、手机号、真实姓名（非授权用例）

发现命中：
1. **停**，不要提交。
2. 报告命中的文件和行（如果内容敏感，只描述位置，别把密钥原样打印给无关上下文）。
3. 处理：删掉/换成占位符/移到环境变量，确认后再提交。`.env`、`*.pem`、`*.key` 直接加进 `.gitignore` 并建议用户 `git rm --cached` 如果之前误提交过。

### 3. 逻辑分组

一个 commit 只做一件事。混合改动拆成多个 commit：

- 功能 A 的代码 + 功能 B 的代码 → 两个 commit
- 修 bug + 顺手重命名 → 分开
- 大规模格式化（如 prettier 全仓跑）→ 单独一个 `style:` commit，别混进功能

用 `git add <具体文件或路径>` 精确暂存，**不要**无脑 `git add .` 或 `git add -A`（除非用户明确要求全提交且你已逐文件看过）。

### 4. 写规范 commit message

用 Conventional Commits 格式：

```
<type>(<scope>): <subject>

<可选：为什么改、怎么改的正文>
```

- type：`feat` `fix` `refactor` `docs` `chore` `perf` `style` `test`
- scope 可有可无：`feat(auth): ...`
- subject：动词开头、祈使句、别超过 50 字符、小写开头
- 有 breaking change：`feat!: ...` 或在正文写 `BREAKING CHANGE: ...`
- 正文解释**为什么**（背景、取舍），别复述代码

### 5. 给用户过目再提交

把将要执行的命令（`git add ... && git commit -m "..."`）展示给用户，确认后执行。执行后看 `git log -1` 和 `git status` 确认提交正确、工作区干净。

## 规则

- 不把未审视的内容加进 commit。
- 不提交二进制/生成物（除非项目惯例如此），提醒用户加 `.gitignore`。
- 敏感信息处理前，任何情况下都先停手。
- 用户说「直接提交」时仍执行扫描，但可以跳过逐条确认。

## 参考

`references/secret-patterns.md` — 完整的敏感模式清单与处理建议。
