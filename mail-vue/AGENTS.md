<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# mail-vue

## Purpose
Vue 3 前端应用，提供邮件管理界面。使用 Element Plus 组件库、Pinia 状态管理、Vue Router 路由、TinyMCE 富文本编辑器。构建后通过 Worker 的 `env.assets.fetch` 提供静态服务。

## Key Files

| File | Description |
|------|-------------|
| `package.json` | 依赖配置（vue 3.5, element-plus 2.9, vite 6.2） |
| `vite.config.js` | Vite 构建配置，自动导入 Element Plus |
| `index.html` | HTML 入口文件 |
| `.env.example` | 环境变量示例 |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `src/` | 源代码目录 (see `src/AGENTS.md`) |
| `public/` | 静态资源（tinymce 编辑器、图片） |

## For AI Agents

### Working In This Directory
- 使用 Vue 3 Composition API + `<script setup>` 语法
- 组件自动导入（unplugin-auto-import + unplugin-vue-components）
- Element Plus 按需导入
- 国际化使用 vue-i18n
- 状态持久化使用 pinia-plugin-persistedstate

### Testing Requirements
```bash
pnpm dev    # 开发服务器 localhost:5173
pnpm build  # 构建生产版本到 dist/
```

### Common Patterns
- 路由定义在 `src/router/index.js`
- 全局状态在 `src/store/` 下的 Pinia stores
- API 请求通过 `src/axios/` 封装的 axios 实例
- 权限指令 `v-perm` 定义在 `src/perm/perm.js`

## Dependencies

### Internal
- `src/init/init.js` - 应用初始化逻辑
- `src/axios/` - HTTP 请求封装
- `src/store/` - Pinia 状态管理

### External
- `vue` 3.5 - 前端框架
- `element-plus` 2.9 - UI 组件库
- `pinia` - 状态管理
- `vue-router` 4.5 - 路由
- `axios` - HTTP 客户端
- `tinymce` - 富文本编辑器
- `dexie` - IndexedDB 封装

<!-- MANUAL: -->
