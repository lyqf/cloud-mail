<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# API Route Handlers

Hono-based REST API route definitions. Each file exports route handlers that integrate with the service layer, user context middleware, and response formatting.

## Architecture

```
Request → Hono Router → Middleware (userContext) → Handler
                                                       ↓
                                                   Service Layer
                                                       ↓
                                                   Response (result.ok/error)
```

**Pattern**: All handlers follow a consistent structure:
- Extract context from request (body via `c.req.json()`, query via `c.req.query()`, params via `c.req.param()`)
- Extract authenticated user ID via `userContext.getUserId(c)` when needed
- Call service method
- Return standardized JSON response via `result.ok(data)` or `c.json(result.ok())`

---

## File Reference

### account-api.js
**Purpose**: Account management for authenticated users.

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/account/list` | List user accounts | ✓ |
| POST | `/account/add` | Create new account | ✓ |
| DELETE | `/account/delete` | Delete account by ID | ✓ |
| PUT | `/account/setName` | Update account name | ✓ |
| PUT | `/account/setAllReceive` | Toggle receive all emails | ✓ |
| PUT | `/account/setAsTop` | Pin account to top | ✓ |

**Service**: `accountService`

---

### all-email-api.js
**Purpose**: Global email operations across all accounts (admin/unrestricted view).

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/allEmail/list` | List all emails system-wide | - |
| GET | `/allEmail/latest` | Get latest emails | - |
| DELETE | `/allEmail/delete` | Physical deletion of emails | - |
| DELETE | `/allEmail/batchDelete` | Batch delete emails | - |

**Service**: `emailService` (allList, allEmailLatest, physicsDelete, batchDelete methods)

---

### analysis-api.js
**Purpose**: Analytics and data visualization endpoints.

| Method | Route | Handler |
|--------|-------|---------|
| GET | `/analysis/echarts` | Generate chart data for visualization |

**Service**: `analysisService`

---

### email-api.js
**Purpose**: User email operations (read, send, list, delete).

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/email/list` | List user's emails | ✓ |
| GET | `/email/latest` | Get latest emails | ✓ |
| GET | `/email/attList` | List attachments | ✓ |
| POST | `/email/send` | Send email | ✓ |
| PUT | `/email/read` | Mark email as read | ✓ |
| DELETE | `/email/delete` | Delete email | ✓ |

**Services**: `emailService`, `attService`

---

### init-api.js
**Purpose**: System initialization (database setup).

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/init/:secret` | Initialize database schema | - |

**Service**: `dbInit.init()`

**Note**: Requires secret parameter in URL path for security.

---

### login-api.js
**Purpose**: Authentication and session management.

| Method | Route | Handler |
|--------|-------|---------|
| POST | `/login` | Authenticate user, return JWT token |
| POST | `/register` | Create new user account |
| DELETE | `/logout` | Invalidate session | ✓ |

**Service**: `loginService`

---

### my-api.js
**Purpose**: Current user profile operations (authenticated context).

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/my/loginUserInfo` | Get current user profile | ✓ |
| PUT | `/my/resetPassword` | Change password | ✓ |
| DELETE | `/my/delete` | Delete own account | ✓ |

**Service**: `userService`

---

### oauth-api.js
**Purpose**: Third-party OAuth authentication.

| Method | Route | Handler |
|--------|-------|---------|
| POST | `/oauth/linuxDo/login` | Linux Do OAuth login |
| PUT | `/oauth/bindUser` | Link OAuth identity to user |

**Service**: `oauthService`

---

### public-api.js
**Purpose**: Public endpoints for unauthenticated operations.

| Method | Route | Handler |
|--------|-------|---------|
| POST | `/public/genToken` | Generate access token |
| POST | `/public/emailList` | List public emails |
| POST | `/public/addUser` | Register new user |

**Service**: `publicService`

---

### r2-api.js
**Purpose**: File storage (R2/S3 object retrieval).

| Method | Route | Handler |
|--------|-------|---------|
| GET | `/oss/*` | Download file by key |

**Features**:
- Wildcard route captures any path under `/oss/`
- Returns object with correct Content-Type header
- Supports content disposition for downloads
- Cache-Control can be set at application level

**Service**: `r2Service`

---

### reg-key-api.js
**Purpose**: Registration key management for invitations/access control.

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| POST | `/regKey/add` | Create registration key | ✓ |
| GET | `/regKey/list` | List registration keys | ✓ |
| GET | `/regKey/history` | View usage history | ✓ |
| DELETE | `/regKey/delete` | Delete registration key | ✓ |
| DELETE | `/regKey/clearNotUse` | Remove unused keys | ✓ |

**Service**: `regKeyService`

---

### resend-api.js
**Purpose**: Webhook receiver for Resend email service events.

| Method | Route | Handler |
|--------|-------|---------|
| POST | `/webhooks` | Process Resend webhooks |

**Response**: Plain text (`success` or error message)

**Service**: `resendService`

**Note**: Webhook endpoint, returns HTTP status 200 on success, 500 on error.

---

### role-api.js
**Purpose**: Role-based access control (RBAC) management.

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/role/list` | List all roles | - |
| GET | `/role/selectUse` | Get available roles | - |
| GET | `/role/permTree` | Get permission hierarchy | - |
| POST | `/role/add` | Create new role | ✓ |
| PUT | `/role/set` | Update role permissions | - |
| PUT | `/role/setDefault` | Set default role | - |
| DELETE | `/role/delete` | Delete role | - |

**Services**: `roleService`, `permService`

---

### setting-api.js
**Purpose**: Application configuration and settings.

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/setting/query` | Get user settings | - |
| GET | `/setting/websiteConfig` | Get public website config | - |
| PUT | `/setting/set` | Update settings | - |
| PUT | `/setting/setBackground` | Upload background image | - |
| DELETE | `/setting/deleteBackground` | Remove background image | - |

**Service**: `settingService`

---

### star-api.js
**Purpose**: Email marking/favoriting system.

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/star/list` | List starred emails | ✓ |
| POST | `/star/add` | Star an email | ✓ |
| DELETE | `/star/cancel` | Remove star from email | ✓ |

**Service**: `starService`

---

### telegram-api.js
**Purpose**: Telegram bot integration for email delivery.

| Method | Route | Handler | Cache |
|--------|-------|---------|-------|
| GET | `/telegram/getEmail/:token` | Retrieve email content for Telegram | 7 days |

**Response**: HTML content with Cache-Control header (public, immutable, 604800s)

**Service**: `telegramService`

---

### test-api.js
**Purpose**: Testing endpoints (empty placeholder).

Currently unused. Reserved for test/debug routes.

---

### user-api.js
**Purpose**: User management (admin operations).

| Method | Route | Handler | Auth |
|--------|-------|---------|------|
| GET | `/user/list` | List all users | ✓ |
| POST | `/user/add` | Create new user | - |
| PUT | `/user/setPwd` | Set user password | - |
| PUT | `/user/setStatus` | Toggle user status | - |
| PUT | `/user/setType` | Change user type | - |
| PUT | `/user/resetSendCount` | Reset sending quota | - |
| PUT | `/user/restore` | Restore user data | - |
| DELETE | `/user/delete` | Physical delete user | - |
| GET | `/user/allAccount` | List all user accounts | - |
| DELETE | `/user/deleteAccount` | Delete account (admin) | - |

**Services**: `userService`, `accountService`

---

## Common Patterns

### Request Body
```javascript
const data = await c.req.json();
// Handler receives: { param1: value, param2: value }
```

### Query Parameters
```javascript
const query = c.req.query();
// Handler receives: { id: '123', filter: 'active' }
```

### URL Parameters
```javascript
const params = c.req.param();
// Handler receives: { token: 'abc123' }
```

### Response Format
```javascript
return c.json(result.ok(data));
// Returns: { code: 0, msg: 'success', data: {...} }

return c.json(result.ok());
// Returns: { code: 0, msg: 'success' }
```

### User Authentication
```javascript
const userId = userContext.getUserId(c);
// Extracts from request context (JWT or session)
```

---

## Middleware Integration

All routes use shared Hono instance from `../hono/hono`. Middleware is configured globally:
- Authentication/authorization
- CORS headers
- Request logging
- Error handling

Routes marked with ✓ in Auth column call `userContext.getUserId(c)`, implying authentication middleware must run first.

---

## Error Handling

Service errors propagate through handlers and are caught by global Hono error middleware. Standard response format:
```javascript
{
  code: non-zero,
  msg: 'error message',
  data: null
}
```

Webhook endpoint (`resend-api.js`) returns plain text status instead of JSON.

---

## Service Layer Mapping

| Service | Routes |
|---------|--------|
| accountService | account-api, user-api |
| emailService | email-api, all-email-api |
| analysisService | analysis-api |
| attService | email-api |
| loginService | login-api |
| userService | user-api, my-api |
| oauthService | oauth-api |
| publicService | public-api |
| r2Service | r2-api |
| regKeyService | reg-key-api |
| resendService | resend-api |
| roleService | role-api |
| permService | role-api |
| settingService | setting-api |
| starService | star-api |
| telegramService | telegram-api |

