<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# src

## Purpose
Cloudflare Worker 后端源代码，包含 API 路由、业务逻辑、数据访问、邮件处理、认证授权等。

## Key Files

| File | Description |
|------|-------------|
| `index.js` | Worker 入口，处理 fetch/email/scheduled 事件 |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `api/` | API 路由定义（Hono routes） (see `api/AGENTS.md`) |
| `service/` | 业务逻辑层 (see `service/AGENTS.md`) |
| `dao/` | 数据访问层（Drizzle ORM schema 和查询） (see `dao/AGENTS.md`) |
| `email/` | 邮件接收处理（Cloudflare Email Routing） (see `email/AGENTS.md`) |
| `hono/` | Hono 应用和中间件配置 (see `hono/AGENTS.md`) |
| `entity/` | 实体类型定义 |
| `model/` | 数据模型 |
| `security/` | 安全相关（JWT、密码加密） |
| `error/` | 错误处理和自定义异常 |
| `init/` | 数据库初始化和迁移 |
| `utils/` | 工具函数 |
| `i18n/` | 国际化配置 |
| `const/` | 常量定义 |
| `template/` | 模板（如邮件模板） |

## For AI Agents

### Working In This Directory
- API 路由在 `api/` 下按功能模块分文件（如 `email-api.js`, `user-api.js`）
- 业务逻辑在 `service/` 中，遵循单一职责原则
- 数据库操作在 `dao/` 中，使用 Drizzle ORM
- 入口文件 `index.js` 处理三种事件：
  - `fetch`: HTTP 请求（API 和静态资源）
  - `email`: 接收邮件（Cloudflare Email Routing）
  - `scheduled`: 定时任务（清理过期数据、重置发送计数）

### Testing Requirements
```bash
pnpm test  # vitest 单元测试
```

### Common Patterns
```javascript
// API 路由定义
export default function (app) {
  app.get('/users', async (c) => {
    const users = await userService.getUsers(c)
    return c.json(users)
  })
}

// Service 层
async function getUsers({ env }) {
  return await dao.getUsers({ env })
}
```

## Dependencies

### Internal
- `dao/` - 数据访问
- `service/` - 业务逻辑
- `security/` - 认证授权
- `error/` - 错误处理

### External
- `hono` - Web 框架
- `drizzle-orm` - ORM
- `resend` - 邮件发送
- `postal-mime` - 邮件解析
- `jsonwebtoken` - JWT

<!-- MANUAL: -->
