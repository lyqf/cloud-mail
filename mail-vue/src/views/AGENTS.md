<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# views

## Purpose
页面视图组件，每个子目录对应一个功能模块的页面。每个视图通过路由配置与 Vue Router 关联。

## Subdirectories

| 目录 | 功能描述 |
|------|---------|
| **404** | 404 错误页面，用户访问不存在的路由时显示 |
| **all-email** | 所有邮件列表，汇总显示全部邮件 |
| **analysis** | 数据分析页面，展示邮件统计和数据报表 |
| **content** | 邮件内容查看，显示单封邮件的完整内容 |
| **draft** | 草稿箱，保存未发送的邮件草稿 |
| **email** | 收件箱，主收箱显示所有接收的邮件 |
| **login** | 登录页面，用户认证和登录表单 |
| **reg-key** | 注册密钥管理，管理邮箱注册密钥 |
| **role** | 角色权限管理，配置用户角色和权限 |
| **send** | 已发送邮件，显示已发出的邮件列表 |
| **setting** | 设置页面，用户个人设置和偏好 |
| **star** | 星标邮件，显示用户标记的重要邮件 |
| **sys-setting** | 系统设置，系统级别的配置管理 |
| **test** | 测试页面，开发测试功能 |
| **user** | 用户管理，管理系统用户 |

## For AI Agents

### Working In This Directory

**文件组织**
- 每个模块为独立目录，内部通常包含 `index.vue` 和可选的子组件
- 使用 Vue 3 Composition API 编写业务逻辑
- 视图组件负责页面布局、数据获取和用户交互

**路由配置**
- 路由在 `/src/router/index.js` 中定义
- 修改视图需要验证对应路由配置是否正确
- 页面通过 `component: () => import('@/views/xxx/index.vue')` 动态导入

**数据流**
1. 视图组件通过 `@/request/` 中的 API 函数获取数据
2. 状态通过 Pinia stores (`@/store/`) 管理
3. 共享组件来自 `@/components/`

### Common Patterns

**API 调用**
```javascript
import { emailList, emailDelete } from "@/request/email.js";
import { starAdd, starCancel } from "@/request/star.js";

// 在组件中使用
const data = await emailList(params);
```

**状态管理**
```javascript
import { useAccountStore } from "@/store/account.js";
import { useEmailStore } from "@/store/email.js";
import { useSettingStore } from "@/store/setting.js";

const emailStore = useEmailStore();
const accountStore = useAccountStore();
const settingStore = useSettingStore();
```

**UI 组件**
- Element Plus 用于表单、按钮、加载态等基础组件
- Iconify Vue 用于图标（如 `material-symbols-light:*`）
- 自定义组件如 `emailScroll` 用于邮件列表滚动加载

**命名约定**
```javascript
defineOptions({
  name: 'email'  // 与目录名对应
})
```

### Testing Requirements

修改视图后验证：
- 访问对应路由页面，检查布局和功能
- 检查数据加载是否正常
- 验证用户交互（点击、滚动、搜索等）有效
- 确认 Element Plus 组件样式无冲突

### Common Issues

**路由导航**
- 使用 `import router from "@/router/index.js"` 和 `router.push()` 进行页面跳转
- 使用 `useRoute()` 获取当前路由信息

**响应式数据**
- 使用 `reactive()` 创建响应式对象
- 使用 `ref()` 创建响应式值
- 使用 `computed()` 创建计算属性

**事件处理**
- 通过 `@event-name` 绑定自定义事件
- 通过 `@click` 等原生事件处理用户操作

## Dependencies

### Internal
- `@/request/*` - API 请求函数（邮件、星标、用户等）
- `@/store/*` - Pinia 状态存储（account、email、setting 等）
- `@/components/*` - 可复用视图组件（email-scroll 等）
- `@/layout/*` - 主布局组件
- `@/utils/*` - 工具函数（时间、转换等）
- `@/router/index.js` - 路由配置

### External
- `element-plus` - UI 组件库
- `vue-router` - 路由管理（`useRoute` 钩子）
- `@iconify/vue` - 图标库
- `nprogress` - 页面加载进度条

### Store Modules
- `useAccountStore()` - 账户信息和认证
- `useEmailStore()` - 邮件数据管理
- `useSettingStore()` - 应用设置和主题
- `useUiStore()` - UI 状态

<!-- MANUAL: -->
