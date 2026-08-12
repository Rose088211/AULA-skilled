---
name: api-contract
description: 设计 REST API 契约并生成 OpenAPI 3 规范（YAML/JSON）。Use whenever the user wants to 设计 API, 接口设计, 定义接口, 生成 OpenAPI, swagger 文档, 前后端联调契约, REST API, 定 endpoint, or wants request/response schemas documented before coding. Triggers even if they only say "帮我把接口定一下" or "写个接口文档".
---

# API 契约设计（REST + OpenAPI 3）

在写实现之前，先把接口的「形状」定下来：资源、路径、动词、状态码、请求/响应 schema。契约定了，前后端就能并行开工，也天然是 OpenAPI 文档的骨架。

## 何时使用

- 要设计新 API、加新端点、定义接口文档。
- 前后端要联调，需要一份双方都认的契约。
- 要把现有接口整理成 OpenAPI 文档给客户端/文档站点用。

## 设计规范

### 资源与路径

- 资源用**复数名词**：`/users`、`/orders`，不用动词（`/getUser`、`/createOrder` 是反模式）。
- 层级用路径参数：`/users/{userId}/orders`。嵌套不超过两层，更深的关系考虑扁平化或查询参数。
- 单个资源用 `{id}`：`GET /users/{userId}`。
- 子资源、集合、动作：能用资源表达的不要用 RPC 动词（`POST /users/{id}/deactivate` 通常可接受，`POST /deactivate-user` 不行）。

### 动词语义

| 动词 | 语义 | 幂等 | 成功响应 |
|---|---|---|---|
| GET | 读取 | 是 | 200 |
| POST | 创建 / 触发动作 | 否 | 201（创建）/ 200 |
| PUT | 整体替换 | 是 | 200 / 204 |
| PATCH | 部分更新 | 否（部分语义） | 200 |
| DELETE | 删除 | 是 | 204 / 200 |

### 状态码（够用即可，别炫技）

`200` 成功 · `201` 创建 · `204` 无内容 · `400` 参数错误 · `401` 未认证 · `403` 无权限 · `404` 不存在 · `409` 冲突（如重复创建）· `422` 业务校验失败 · `429` 限流 · `500` 服务端错误。

### 统一约定

- **分页**：`GET /users?page=1&page_size=20` → 响应 `{ "items": [...], "total": 128, "page": 1, "page_size": 20 }`。
- **错误格式**统一：`{ "error": { "code": "USER_NOT_FOUND", "message": "用户不存在" } }`，`code` 是给程序用的稳定标识，`message` 给人看。
- **时间**一律 ISO 8601 UTC（`2026-08-13T09:00:00Z`），命名带 `_at`。
- **ID** 用字符串（UUID 或雪花），不暴露自增数字。
- 字段名：API 层用 `snake_case`（对外稳定），或与团队约定保持一致，一旦发布不轻易改。

## 流程

1. **确认需求**：这个接口给谁用、做什么、关键场景。问清楚再动手，别直接开写。
2. **列资源**：把名词圈出来（用户、订单、文章……），确定哪些需要 API。
3. **定端点**：每个资源 × 需要的动词，填成表格（方法/路径/用途/鉴权）。
4. **定 schema**：请求体、响应体、错误体。字段能少则少，能可选就可选。
5. **生成 OpenAPI 3**：用下面的模板产出 `openapi.yaml`，放 `docs/` 或项目约定的位置。
6. **校验**：`npx --yes @redocly/cli lint openapi.yaml` 或 `npx swagger-cli validate`。
7. **过一遍给用户**：特别是状态码、字段命名、breaking 风险点。

## OpenAPI 模板

直接复用 `references/openapi-template.yaml`，改 `info`、`paths`、`schemas` 三处即可。

## 规则

- 契约先于实现：先定接口，再谈代码。
- 别定义用不上的端点——每个端点都要有真实的消费方。
- 版本策略提前约定（URL 前缀 `/v1` 或 header），并写进文档。
- 生成的文档要能通过 lint，不能留语法错误。

## 参考

`references/openapi-template.yaml` — 可直接改写的 OpenAPI 3.0 模板。
