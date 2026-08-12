---
name: changelog-generator
description: 从 git 历史生成或更新 CHANGELOG.md，遵循 Keep a Changelog 与语义化版本规范。Use whenever the user asks for 更新 CHANGELOG, 生成更新日志, release notes, 这次发布了什么, what changed since the last release, bump version, 版本号该升到几, or wants a human-readable summary of commits since a tag/branch point. Triggers even when they only say "总结一下这周的改动".
---

# Changelog 生成器

把 git 历史整理成面向读者的 CHANGELOG.md。读者是使用这个项目的人，不是维护者——所以他们要的是「有什么变化」，不是「第 483 个 commit 修了个 typo」。

## 何时使用

- 用户要生成/更新 CHANGELOG.md、release notes、发布说明。
- 用户问「这次版本改了什么」「这周改了什么」「上一个 tag 之后有什么变化」。
- 用户要判断下一个版本号该怎么升。

## 流程

### 1. 确定范围

- 有 tag：`git tag --sort=-v:refname | head` 找最近的 tag，范围为 `<last-tag>..HEAD`。
- 没 tag：若已有 CHANGELOG.md，范围是 `Unreleased` 章节上次记录的点；否则从最早的 commit 开始。
- 明确是分支对比：用 `git log main..branch`（方向问清：这是要合入 main 的改动）。

### 2. 拉取并分类

```
git log <since>..HEAD --pretty=format:"%h %s%n%b" 
```

把每条按 Keep a Changelog 的六个章节归类：

| 章节 | 含义 | 常见 prefix |
|---|---|---|
| Added | 新功能 | feat, add, new |
| Changed | 已有功能的改动 | change, refactor(行为变), update |
| Fixed | 修 bug | fix, bugfix, hotfix |
| Removed | 删除功能 | remove, drop, deprecate+删 |
| Deprecated | 即将废弃 | deprecate |
| Security | 安全修复 | security, CVE |

`docs`/`chore`/`test`/`style`/`ci` 默认不写进用户向的 changelog，除非有面向用户的影响。

### 3. 归纳成面向用户的语言

- **合并同类**：5 条「修了登录页 xxx 的小问题」→ 1 条「修复登录页的若干显示问题」。
- **去掉内部行话**：commit 里的「重构 xxx 模块」对用户没有意义，改成用户能感知的变化，或者不写。
- **一句话一条**，动词开头，别复读 commit message 原文。
- 保留 breaking change，单独列出，并写清楚「怎么迁移」。

### 4. 定版本号（语义化版本）

没有 tag 时基于分类推断：
- 有 breaking change 或大功能重构 → `X+1.0.0`（主版本）
- 有新功能（Added）→ `0.x.0` 或 `X.Y+1.0`（次版本）
- 只有修复/小改动 → `X.Y.Z+1`（补丁）

已有 tag 时，比较 `last-tag` 与 `next-tag` 判断怎么升，并把对应章节移入新版本。别把 `Unreleased` 清空——升级后新开一个 `[Unreleased]` 占位。

### 5. 写入 / 更新 CHANGELOG.md

顶部模板（Keep a Changelog 风格）：

```markdown
# Changelog

本项目所有重要变更都记录在此文件。
格式遵循 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，
版本号遵循 [语义化版本](https://semver.org/lang/zh-CN/)。

## [Unreleased]

### Added
### Changed
### Fixed
```

每个版本章节按时间倒序排列，新的在上。如果仓库还没有 CHANGELOG.md，创建它；如果有，只在对应版本/Unreleased 章节追加，别推倒重来。

## 规则

- 不改动 commit 的事实：是否 breaking、是否删了功能，以代码为准，别凭 message 猜。
- 不确定某条算哪类，宁可放在 Changed 也别删掉。
- 生成后给用户看一遍，特别是 breaking change 列表。
- 若用户要求「包括所有 commit 明细」，可以额外给一份完整清单，但默认保持精炼。

## 示例

> 输入范围：`v1.2.0..HEAD`，包含这些改动：
> - `feat: 新增导出 CSV 功能`
> - `fix: 修复导出时日期格式错误`
> - `refactor: 重构数据层`
> - `chore: 更新依赖`
> - `fix: 修复登录偶发白屏`
> - `feat!: 配置项 API 由 JSON 改为 TOML`

> 输出：
> ```markdown
> ## [v1.3.0] - 2026-08-13
> 
> ### Added
> - 支持将数据导出为 CSV
> 
> ### Fixed
> - 修复导出时日期格式错误的问题
> - 修复登录页偶发白屏
> 
> ### 破坏性变更
> - 配置文件格式由 JSON 改为 TOML，升级前请迁移现有配置。
> ```
