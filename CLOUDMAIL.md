# CloudMail 临时邮箱使用说明

## 原理

使用自建 cloud-mail 服务的 admin 账号，通过 `/account/add` 接口批量创建邮箱别名。
每次注册时动态创建一个 `oc{随机10位}@llmailab.xyz` 的别名，用完即弃。

## API 接口

### 认证
- Header 名：`Authorization`
- Header 值：直接传 JWT token（**不需要** `Bearer ` 前缀）

### 登录获取 token
```
POST /api/login
{"email": "admin@llmailab.xyz", "password": "mikejson123321"}
→ data.token
```

### 创建别名邮箱
```
POST /api/account/add
{"email": "oc{random}@llmailab.xyz"}
→ data.accountId
```
- admin 账号无数量限制
- auth_credential 格式：`{jwt_token}|{accountId}`

### 轮询邮件
```
GET /api/email/list?accountId={id}&type=0&size=10&timeSort=0
```
- `type=0` 表示收件（数字，不是字符串）
- `timeSort=0` 按时间倒序（最新的在前）
- 邮件内容字段：`content`（HTML），`subject`，`sendEmail`

### 验证码提取
邮件 `content` 字段为 HTML，验证码位于：
```html
<p style="...background-color: #F3F3F3...">576135</p>
```
`_extract_code()` 可正确提取。

## sync_config.json 配置
```json
{
  "mail_provider": "cloudmail",
  "mail_providers": ["cloudmail"],
  "mail_provider_configs": {
    "cloudmail": {
      "api_base": "https://llmailab.xyz/api",
      "admin_email": "admin@llmailab.xyz",
      "admin_password": "mikejson123321",
      "domain": "llmailab.xyz"
    }
  }
}
```

## 已知问题与修复历史
- `type` 字段必须传数字 `0`，传字符串 `"RECEIVE"` 返回空列表
- `Authorization` header 直接传 token，不加 `Bearer ` 前缀
- token 过期时（401）自动重新登录刷新
