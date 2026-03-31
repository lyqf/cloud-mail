<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# mail-worker

## Purpose
Cloudflare Worker 后端服务，处理邮件收发、用户认证、权限控制、附件管理。使用 Hono Web 框架，Drizzle ORM 操作 D1 数据库。

## Key Files

| File | Description |
|------|-------------|
| `package.json` | 依赖配置（hono 4.7, drizzle-orm 0.37） |
| `wrangler.toml` | Cloudflare Worker 配置模板 |
| `wrangler-action.toml` | GitHub Actions 部署配置模板 |
| `drizzle.config.js` | Drizzle ORM 配置 |
| `vitest.config.js` | Vitest 测试配置 |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `src/` | 源代码目录 (see `src/AGENTS.md`) |
| `test/` | 单元测试 |

## For AI Agents

### Working In This Directory
- 使用 Hono 框架定义路由和中间件
- Drizzle ORM 操作 D1 数据库（schema 在 `src/dao/schema.js`）
- JWT 认证 + KV 会话存储
- 邮件发送使用 Resend API
- 邮件接收通过 `email` handler（Cloudflare Email Routing）
- 附件存储在 R2

### Testing Requirements
```bash
pnpm test           # vitest 单元测试
pnpm deploy         # 部署到 Cloudflare Workers
pnpm wrangler dev   # 本地开发（需要 wrangler 配置）
```

### Common Patterns
- 路由定义在 `src/api/` 下按功能模块分组
- 业务逻辑在 `src/service/` 中
- 数据访问在 `src/dao/` 中
- 错误处理在 `src/error/` 中
- 实体类型在 `src/entity/` 中

## Dependencies

### Internal
- `src/init/` - 数据库初始化和迁移
- `src/dao/` - 数据访问层（Drizzle ORM）
- `src/service/` - 业务逻辑层
- `src/api/` - API 路由层

### External
- `hono` 4.7 - Web 框架
- `drizzle-orm` 0.37 - ORM
- `resend` - 邮件发送 API
- `postal-mime` - 邮件解析
- `jsonwebtoken` - JWT 认证

<!-- MANUAL: -->
