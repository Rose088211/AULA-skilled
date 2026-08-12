# 敏感信息扫描模式清单

提交前对全部改动（含新增的 untracked 文件）做一次搜索。下面按风险分级，命中即停手。

## 高优先级——绝对不该进 git

**密钥 / 令牌**
- AWS Access Key：`AKIA[0-9A-Z]{16}`
- GitHub / GitLab 令牌：`ghp_[A-Za-z0-9]{36}`、`github_pat_`、`glpat-[A-Za-z0-9_\-]{20}`
- Anthropic / OpenAI 类：`sk-[A-Za-z0-9]{20,}`（注意 `sk-` 也可能是普通配置，需人工确认）
- JWT：`eyJ[A-Za-z0-9_-]{10,}\.eyJ[A-Za-z0-9_-]{10,}`
- Stripe 等：`sk_live_`、`pk_live_`
- 私钥块：`-----BEGIN (RSA|EC|OPENSSH|DSA) PRIVATE KEY-----`
- 云厂商 Token：`AKIA`、`xox[baprs]-`（Slack）、`ya29\.`（Google OAuth）

**数据库 / 连接串**
- 含密码的 URL：`[a-z]+://[^:/@\s]+:[^@\s]+@` （如 `postgres://user:pass@host`）
- Redis/MySQL/Mongo 裸连接串

**环境变量真值**
- `.env`、`.env.production` 等文件本身（无论内容），若必须提交则只提交 `.env.example`
- `password`、`passwd`、`secret`、`token`、`api[_-]?key` 赋了非占位符的真值

## 中优先级——视项目而定

- 内网 / 私有 IP：`(10\.|172\.(1[6-9]|2\d|3[01])\.|192\.168\.)`
- 真实用户数据：姓名、手机号、身份证号、邮箱，出现在示例之外的硬编码
- 云厂商资源 ID、内网域名、带端口的内部服务地址

## 处理方式

1. **停下来**，不提交。
2. 告诉用户命中的文件位置（不打印密钥原文）。
3. 按情况处理：
   - 误加的真密钥 → 从文件删掉 → **同时考虑该密钥已泄露**，建议轮换密钥，而不是只删文件。
   - 该用环境变量的 → 改成读取环境变量，提交 `.env.example`。
   - 历史里已提交过密钥 → `git rm --cached` + 建议 `git filter-repo` / 轮换密钥（远程可能已同步）。
4. 把 `.env`、`*.pem`、`*.key`、`*.p12` 加进 `.gitignore`。

## 搜什么文件

- 所有新增/修改的文本文件，不只是 `.env` 类。
- 配置文件、示例代码、README 里的代码块、测试里的 fixture（测试里最容易藏真凭据）。
- JSON/YAML/TOML/ini 里的 `key`、`token`、`secret`、`credential` 字段。
