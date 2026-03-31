<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# src

## Purpose
Vue 3 前端应用源代码，包含组件、视图、路由、状态管理、API 请求、工具函数等。

## Key Files

| File | Description |
|------|-------------|
| `main.js` | 应用入口，初始化 Vue、Pinia、Router、i18n |
| `App.vue` | 根组件，配置 Element Plus locale |
| `style.css` | 全局样式 |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `views/` | 页面视图组件 (see `views/AGENTS.md`) |
| `components/` | 可复用组件 (see `components/AGENTS.md`) |
| `layout/` | 布局组件（header, aside, main, account, write） (see `layout/AGENTS.md`) |
| `router/` | Vue Router 路由配置 |
| `store/` | Pinia 状态管理 stores |
| `axios/` | Axios 实例和拦截器配置 |
| `request/` | API 请求函数封装 |
| `utils/` | 工具函数 |
| `i18n/` | 国际化配置（中文/英文） |
| `perm/` | 权限指令和权限检查 |
| `db/` | IndexedDB 数据库封装（Dexie） |
| `icons/` | 图标注册 |
| `enums/` | 枚举常量 |
| `echarts/` | ECharts 图表配置 |
| `init/` | 应用初始化逻辑 |

## For AI Agents

### Working In This Directory
- 页面组件放在 `views/` 下，按功能模块分目录
- 可复用组件放在 `components/` 下
- API 请求统一通过 `request/` 中的函数调用
- 全局状态使用 Pinia stores（`store/`）
- 路由配置在 `router/index.js`

### Testing Requirements
- 修改后运行 `pnpm dev` 查看效果
- 检查浏览器控制台是否有错误

### Common Patterns
```javascript
// 使用 Composition API
<script setup>
import { ref, computed } from 'vue'
import { useRouter } from 'vue-router'
import { useUserStore } from '@/store/user.js'

const router = useRouter()
const userStore = useUserStore()
</script>
```

## Dependencies

### Internal
- `axios/` - HTTP 请求
- `store/` - 状态管理
- `router/` - 路由
- `utils/` - 工具函数

### External
- `vue` - 框架核心
- `element-plus` - UI 组件
- `pinia` - 状态管理
- `vue-router` - 路由
- `axios` - HTTP 客户端

<!-- MANUAL: -->
