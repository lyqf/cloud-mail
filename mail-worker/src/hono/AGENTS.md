<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# hono

## Purpose
Hono 应用初始化、中间件配置、全局错误处理、CORS 跨域资源共享设置。所有 API 路由在此处注册和导出。

## Key Files

| File | Description |
|------|-------------|
| `hono.js` | Hono 应用实例、CORS 中间件、全局错误处理器 |
| `webs.js` | 应用入口，导入所有 API 路由模块 |

## For AI Agents

### Working In This Directory

**hono.js - 应用核心配置**
- 创建 Hono 应用实例：`const app = new Hono()`
- CORS 中间件：`app.use('*', cors())` 允许跨域请求
- 全局错误处理：`app.onError()` 统一捕获异常
  - 检测 BizError（业务错误）进行区分处理
  - 检测 KV 数据库未绑定（502 错误）
  - 检测 D1 数据库未绑定（502 错误）
  - 返回统一错误响应格式

**webs.js - 路由注册入口**
- 导入 Hono 应用实例
- 导入安全模块（security）
- 顺序导入所有 API 模块，自动注册路由：
  - `email-api` - 邮件相关接口
  - `user-api` - 用户相关接口
  - `login-api` - 登录认证接口
  - `setting-api` - 用户设置接口
  - `account-api` - 账户管理接口
  - `star-api` - 收藏标星接口
  - `test-api` - 测试接口
  - `r2-api` - R2 对象存储接口
  - `resend-api` - Resend 邮件服务接口
  - `my-api` - 个人信息接口
  - `role-api` - 角色权限接口
  - `all-email-api` - 全局邮件接口
  - `init-api` - 初始化接口
  - `analysis-api` - 数据分析接口
  - `reg-key-api` - 注册密钥接口
  - `public-api` - 公开接口
  - `telegram-api` - Telegram 集成接口
  - `oauth-api` - OAuth 认证接口
- 导出应用实例给 Worker 使用

### Error Handling Strategy

**全局错误处理流程**
1. 任何路由抛出异常 → `app.onError()` 捕获
2. 判断错误类型和消息
3. 特殊处理数据库绑定错误（返回 502）
4. 使用 `result.fail()` 返回统一格式错误响应
5. 所有错误通过 console 日志记录（BizError 用 console.log，其他错误用 console.error）

### Integration Points

**与 API 模块的关系**
- `webs.js` 是注册点，所有 API 模块通过 import 语句自动在此处注册
- API 模块应导出默认函数或自执行代码来注册路由到全局 app
- 模块加载顺序决定路由优先级（先定义的优先匹配）

**与 Worker 入口的关系**
- `webs.js` 导出的 app 实例被 Worker 的 `index.js` 使用
- app 处理所有 HTTP 请求的路由分发

### Common Patterns

**添加新的 API 模块**
1. 在 `api/` 目录创建新的 API 文件（如 `new-feature-api.js`）
2. 在 `webs.js` 中添加 import 语句
3. API 文件应自动注册路由到全局 app 实例

**错误处理最佳实践**
- 业务逻辑中抛出 BizError 会被自动捕获并返回错误信息
- 数据库未绑定的错误会被检测并返回 502
- 其他异常会被记录并返回通用错误响应

## Dependencies

### Internal
- `../model/result` - 统一响应格式（包含 fail() 方法）
- `../security/security` - 安全认证和授权
- `../api/*` - 所有 API 路由模块

### External
- `hono` - Web 框架
- `hono/cors` - CORS 中间件

<!-- MANUAL: -->
