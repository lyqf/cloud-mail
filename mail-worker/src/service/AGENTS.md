<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# service

## Purpose
业务逻辑层，处理复杂的业务规则、流程控制和数据操作。services 与 dao 层隔离，services 调用 dao 进行数据访问。所有业务验证、权限检查、流程编排在此层进行。

## Architecture

```
[API routes]
    ↓
[Service layer] ← 业务逻辑、验证、流程控制
    ↓
[DAO layer] ← 数据访问
```

Services 可调用其他 services，但禁止循环依赖。

## Key Files

| File | Lines | Description |
|------|-------|-------------|
| `email-service.js` | 819 | 邮件业务逻辑：收发、草稿、搜索、转发、归档、删除 |
| `user-service.js` | 379 | 用户管理：登录信息、配置、统计、分页 |
| `account-service.js` | 271 | 账户管理：添加、删除、启用/禁用、绑定验证 |
| `login-service.js` | 269 | 登录认证：密码登录、邮件验证、会话管理 |
| `att-service.js` | 264 | 附件处理：上传、下载、清理、内嵌资源管理 |
| `role-service.js` | 224 | 角色权限：权限列表、分配、更新 |
| `setting-service.js` | 212 | 平台设置：邮件发送配置、注册政策、速率限制 |
| `public-service.js` | 195 | 公开服务：无认证访问的功能 |
| `oauth-service.js` | 119 | OAuth 认证：第三方登录、令牌管理 |
| `reg-key-service.js` | 102 | 注册密钥管理：生成、验证、使用 |
| `s3-service.js` | 93 | S3 兼容存储服务封装 |
| `telegram-service.js` | 91 | Telegram 推送通知 |
| `verify-record-service.js` | 90 | 验证记录：邮件验证、验证码管理 |
| `analysis-service.js` | 90 | 数据分析：统计图表、趋势数据 |
| `star-service.js` | 83 | 星标邮件管理 |
| `r2-service.js` | 66 | Cloudflare R2 存储服务 |
| `resend-service.js` | 50 | Resend 邮件发送 API 封装 |
| `perm-service.js` | 37 | 权限检查：用户权限验证 |
| `kv-obj-service.js` | 36 | Cloudflare KV 存储操作 |
| `turnstile-service.js` | 35 | Cloudflare Turnstile 验证码 |

## Service Dependencies

### Core Services (Most Referenced)
- **email-service**: 依赖 account, user, role, setting, att, star, telegram services
- **account-service**: 依赖 user, email, setting, verify-record, turnstile, role services
- **user-service**: 依赖 account, email, perm, role, oauth services
- **setting-service**: 被多个 services 依赖，核心配置来源
- **login-service**: 依赖 user, account, oauth services

### Storage Services
- **att-service**: 管理附件，依赖 r2-service, setting-service
- **r2-service**: R2 存储操作
- **s3-service**: S3 兼容存储
- **kv-obj-service**: Cloudflare KV 操作（会话、缓存）

### Support Services
- **role-service**: 角色权限管理
- **perm-service**: 权限验证
- **verify-record-service**: 验证记录
- **reg-key-service**: 注册密钥
- **star-service**: 星标管理
- **analysis-service**: 数据统计
- **oauth-service**: OAuth 认证
- **telegram-service**: Telegram 推送
- **resend-service**: 邮件发送 API
- **turnstile-service**: Turnstile 验证
- **public-service**: 公开功能

## Common Patterns

### Service 方法签名
```javascript
// 标准 service 对象导出
const serviceNameService = {
  async methodName(c, params, userId) {
    // c: Hono context (包含 c.env 环境变量)
    // params: 请求参数对象
    // userId: 可选，当前用户 ID
    return result;
  }
}
export default serviceNameService;
```

### 业务验证流程
```javascript
async add(c, params, userId) {
  // 1. 获取配置
  const { setting1, setting2 } = await settingService.query(c);

  // 2. 参数验证
  if (!params.field) {
    throw new BizError(t('emptyField'));
  }

  // 3. 业务规则验证
  if (!verifyUtils.isEmail(params.email)) {
    throw new BizError(t('invalidEmail'));
  }

  // 4. 权限检查
  const perm = await permService.checkPermission(c, userId, 'perm_key');

  // 5. 数据操作
  const result = await dao.insert(c, data);

  // 6. 返回结果
  return result;
}
```

### 跨 Service 调用
```javascript
// 获取用户完整信息：调用多个 service
async getUserProfile(c, userId) {
  const [user, account, role, perms] = await Promise.all([
    userService.selectById(c, userId),
    accountService.selectByEmail(c, user.email),
    roleService.selectById(c, user.type),
    permService.userPermKeys(c, userId)
  ]);
  return { user, account, role, perms };
}
```

### 错误处理
```javascript
import BizError from '../error/biz-error';

// 业务错误（用户可见）
throw new BizError(t('errorMessage'));

// HTTP 错误码
throw new BizError(t('unauthorized'), 401);
throw new BizError(t('forbidden'), 403);
```

### 国际化
```javascript
import { t } from '../i18n/i18n';

throw new BizError(t('keyName'));
throw new BizError(t('keyWithParam', { param: value }));
```

## For AI Agents

### Working In This Directory

1. **添加新 Service**
   - 创建 `feature-service.js` 文件
   - 定义 `const featureService = { ... }`
   - 所有方法签名 `async method(c, params, userId)`
   - 导出：`export default featureService`

2. **调用其他 Service**
   ```javascript
   import otherService from './other-service';

   async myMethod(c, params, userId) {
     const data = await otherService.getData(c, params);
     return process(data);
   }
   ```

3. **参数验证**
   - 使用 `../utils/verify-utils` 提供的验证函数
   - 参数为空或格式错误时抛 `BizError`
   - 业务规则冲突时抛 `BizError`

4. **权限检查**
   - 调用 `permService.checkPermission()` 验证权限
   - 调用 `permService.userPermKeys()` 获取用户权限列表

5. **数据访问**
   - 通过 DAO 层读写，不直接操作数据库
   - DAO 在 `../dao/` 下，遵循数据访问隔离

6. **异步操作**
   - 多个独立操作使用 `Promise.all()`
   - 顺序依赖使用 `await` 链式调用

### Testing Requirements
```bash
pnpm test  # vitest 单元测试
```

### Dependency Rules
- Services 可互相调用（避免循环依赖）
- 禁止直接操作数据库（使用 DAO 层）
- 禁止在 service 中定义路由（路由在 `api/` 中）
- 禁止直接访问 HTTP（使用 context 参数）

## Dependencies

### Internal
- `../entity/` - 实体和 ORM
- `../dao/` - 数据访问
- `../error/` - 错误处理
- `../utils/` - 工具函数
- `../const/` - 常量
- `../i18n/` - 国际化
- `../security/` - 认证授权

### External
- `drizzle-orm` - 数据库查询
- `resend` - 邮件 API
- `dayjs` - 时间处理
- `linkedom` - HTML 解析
- `uuid` - UUID 生成

### Cloudflare
- `c.env.db` - D1 数据库
- `c.env.kv` - KV 存储
- `c.env.r2` - R2 对象存储
- `c.env.env` - 环境变量（admin, domain, resendKey 等）

<!-- MANUAL: -->
