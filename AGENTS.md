<!-- Generated: 2026-03-18 -->

# Cloud Mail

## Purpose
基于 Cloudflare 的邮箱服务，支持邮件收发、附件、推送、RBAC 权限、数据可视化。前后端分离架构，部署到 Cloudflare Workers，使用 D1/KV/R2 作为存储层。

## Key Files

| File | Description |
|------|-------------|
| `README.md` | 项目说明与功能介绍 |
| `LICENSE` | MIT 许可证 |
| `.gitignore` | Git 忽略规则 |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `mail-vue/` | Vue 3 前端应用 (see `mail-vue/AGENTS.md`) |
| `mail-worker/` | Cloudflare Worker 后端服务 (see `mail-worker/AGENTS.md`) |
| `doc/` | 文档与截图资源 |
| `.github/` | GitHub Actions CI/CD 配置 |

## For AI Agents

### Architecture Overview
- **Frontend**: Vue 3 + Element Plus + Vite，构建产物通过 Worker 的 `env.assets.fetch` 提供静态服务
- **Backend**: Hono Web 框架运行在 Cloudflare Worker 上
- **Storage**: D1 (SQL, Drizzle ORM), KV (缓存/会话), R2 (附件/文件)
- **Email Send**: Resend API
- **Email Receive**: Cloudflare Email Routing → Worker `email` handler
- **Auth**: JWT + KV 会话存储，RBAC 权限控制
- **Deploy**: GitHub Actions → Wrangler 部署

### API Convention
- 所有 API 路径以 `/api/` 开头，在 `index.js` 中被 strip 为 `/` 后交给 Hono 处理
- 静态资源路径 `/static/` 和 `/attachments/` 通过 KV 对象服务响应

### Testing
```bash
cd mail-worker && pnpm test   # vitest
cd mail-vue && pnpm dev       # 本地开发
```

### Deploy
```bash
cd mail-worker && pnpm deploy  # wrangler deploy
```

<!-- MANUAL: -->
