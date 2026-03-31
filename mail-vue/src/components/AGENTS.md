<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# Components

## Purpose
可复用 Vue 3 组件库，包含邮件列表、编辑器、加载动画、菜单等功能组件。所有组件基于 Element Plus UI 框架、Iconify 图标库、TinyMCE 富文本编辑器等外部依赖。

## Subdirectories & Components

| Directory | Component | Description |
|-----------|-----------|-------------|
| `email-scroll/` | EmailScroll | 虚拟滚动邮件列表，支持骨架屏加载、多选、右键菜单、星标操作 |
| `email-scroll/skeleton/` | SkeletonBlock | 邮件列表骨架屏，加载态占位符 |
| `hamburger/` | Hamburger | 汉堡菜单图标，支持旋转激活态 |
| `loading/` | Loading | 四点旋转加载动画，支持自定义大小 |
| `send-percent/` | SendPercent | 邮件发送进度显示，含加载图标和百分比文本 |
| `shadow-html/` | ShadowHtml | Shadow DOM HTML 渲染器，支持自动缩放和样式隔离 |
| `tiny-editor/` | TinyEditor | TinyMCE 富文本编辑器封装，支持主题切换、多语言 |

## Component API Reference

### EmailScroll
邮件列表虚拟滚动组件，支持大列表高性能渲染。

**Props:**
- `getEmailList: Function` - 获取邮件列表的异步函数，签名 `(emailId, size) => Promise<{list, total, latestEmail}>`
- `emailDelete: Function` - 删除邮件函数
- `emailRead: Function` - 标记邮件已读函数
- `starAdd: Function` - 添加星标函数
- `starCancel: Function` - 取消星标函数
- `cancelSuccess: Function` - 星标取消成功回调
- `starSuccess: Function` - 星标添加成功回调
- `actionLeft: String` - 左侧操作栏左填充（default: '0'）
- `timeSort: Number` - 时间排序标志（default: 0）
- `showStatus: Boolean` - 显示邮件状态图标（default: false）
- `showAccountIcon: Boolean` - 显示账号切换图标（default: true）
- `showUserInfo: Boolean` - 显示用户信息行（default: false）
- `showStar: Boolean` - 显示星标按钮（default: true）
- `allowStar: Boolean` - 允许星标操作（default: true）
- `type: String` - 组件类型，支持 'email', 'star', 'send', 'all-email', 'draft'（default: 'email'）
- `showFirstLoading: Boolean` - 首次加载时显示骨架屏（default: true）
- `showUnread: Boolean` - 显示未读标记（default: false）

**Emits:**
- `@jump` - 点击邮件项时触发，参数为邮件对象
- `@refresh-before` - 刷新前触发
- `@delete-draft` - 草稿删除时触发
- `@right-search` - 右键菜单搜索时触发

**Methods (expose):**
- `refreshList()` - 刷新邮件列表
- `deleteEmail(emailIds)` - 删除邮件
- `addItem(email)` - 添加邮件项
- `handleList(list)` - 处理邮件列表数据

**Slots:**
- `#first` - 头部左侧操作栏插槽
- `#name` - 邮件发件人名称插槽
- `#subject` - 邮件主题插槽

### SkeletonBlock
邮件列表加载骨架屏组件。

**Props:**
- `rows: Number` - 骨架行数（default: 1）
- `showStar: Boolean` - 显示星标占位符（default: true）
- `accountShow: Boolean` - 显示账号列占位符（default: false）
- `showStatus: Boolean` - 显示状态图标占位符（default: false）
- `showUserInfo: Boolean` - 显示用户信息占位符（default: false）
- `type: String` - 组件类型（default: ''）

### Hamburger
汉堡菜单图标组件。

**Props:**
- `isActive: Boolean` - 激活态，控制图标旋转（default: false）

**Emits:**
- `@toggleClick` - 点击时触发

### Loading
旋转加载动画组件。

**Props:**
- `size: Number` - 加载动画大小（px）（default: 30）

### SendPercent
邮件发送进度显示组件。

**Props:**
- `value: Number | String` - 进度百分比
- `desc: String` - 进度描述文本

### ShadowHtml
Shadow DOM HTML 渲染组件，用于隔离邮件 HTML 内容。

**Props:**
- `html: String` - HTML 内容（required）

**Features:**
- 自动提取 `<body style="">` 属性并应用到内容容器
- 移除脚本、样式、媒体标签以防止 XSS
- 支持图片自适应缩放
- 提供全局字体和颜色样式
- Shadow DOM 隔离防止样式污染

### TinyEditor
TinyMCE 富文本编辑器包装组件。

**Props:**
- `defValue: String` - 默认内容（default: ''）
- `editorId: String` - 编辑器唯一标识（default: `editor-${Date.now()}`）

**Emits:**
- `@change` - 内容变化时触发，参数 `(content, text)`
- `@focus` - 编辑器获得焦点时触发

**Methods (expose):**
- `clearEditor()` - 清空编辑器内容
- `focus()` - 编辑器获得焦点
- `getContent()` - 获取编辑器内容

**Features:**
- 动态加载 TinyMCE 脚本（`/tinymce/tinymce.min.js`）
- 主题跟随应用深色模式（`oxide` | `oxide-dark`）
- 多语言支持（中文 `zh_CN` | 英文 `en`）
- 工具栏：文本格式、颜色、列表、链接、图片、表格、代码预览、全屏
- 图片上传转 Base64 Blob
- 自定义内容样式（`/tinymce/css/index.css`）

## Design Principles

### Virtual Scrolling
- EmailScroll 使用 VueUse 的 `UseVirtualList` 实现高性能虚拟滚动
- 适合大量邮件列表渲染（itemHeight 自适应）
- 支持响应式高度调整（1367px 为 PC/Mobile 分界点）

### Shadow DOM Isolation
- ShadowHtml 采用 Shadow DOM API 隔离邮件 HTML 样式
- 防止邮件内容 CSS 污染应用全局样式
- 使用 `:host` 伪元素重置默认样式

### Skeleton Loading
- 骨架屏与实际组件布局一致
- 支持多行加载态
- 减少加载感知时间

### Editor Integration
- TinyEditor 动态加载 TinyMCE 以减少初始包体积
- 支持主题/语言动态切换（destroy + reinit）
- Base64 图片嵌入避免上传依赖

## For AI Agents

### Component Usage Pattern
```vue
<!-- EmailScroll 示例 -->
<template>
  <EmailScroll
    :getEmailList="fetchEmails"
    :emailDelete="deleteEmails"
    :starAdd="addStar"
    @jump="handleJump"
    type="all-email"
    show-user-info
  />
</template>

<!-- TinyEditor 示例 -->
<template>
  <TinyEditor
    ref="editorRef"
    def-value="<p>Draft content</p>"
    @change="handleEditorChange"
  />
</template>
```

### State Management Integration
- 组件通过 props 接收回调函数，与 Pinia store 连接
- 使用 `emits` 向父级通知事件
- EmailScroll 监听 `emailStore.deleteIds` 等 store 变化以同步删除/星标状态

### Common Issues & Solutions

**EmailScroll 虚拟滚动跳闪：**
- 设置 `overscan: 15` 预加载缓冲区
- 使用 `:key="keyCount"` 强制刷新（itemHeight 变化时）

**TinyEditor 白屏：**
- 检查 `/public/tinymce/tinymce.min.js` 路径
- 确保 `uiStore.dark` 值有效

**ShadowHtml 缩放不生效：**
- 父容器必须设置固定宽度
- 检查 HTML 内容是否包含 `<body style="">`

### Testing Checklist
- [ ] 组件在生产构建后仍可正常工作
- [ ] 响应式断点 1366px 布局切换正确
- [ ] 国际化文本更新时组件重新渲染
- [ ] 深色模式切换无闪烁
- [ ] 虚拟列表滚动到底部触发加载

## Dependencies

### External Libraries
- `tinymce` - 富文本编辑器（通过 `/public/tinymce/` 提供）
- `element-plus` - UI 组件库（Checkbox, Dropdown, Empty, Skeleton, Tag, Tooltip）
- `@iconify/vue` - 图标库（ion:reload, uiw:delete, solar:star-line-duotone 等）
- `@vueuse/components` - VueUse 组件（UseVirtualList）
- `@vueuse/core` - VueUse hooks（useScroll）
- `vue-i18n` - 国际化（i18n 翻译键）

### Internal Stores
- `@/store/email.js` - 邮件状态（deleteIds, addStarEmailId, cancelStarEmailId）
- `@/store/ui.js` - UI 状态（dark, accountShow, writerRef）
- `@/store/setting.js` - 应用设置（lang, manyEmail）

### Utilities
- `@/utils/time-utils.js` - `sleep` 延迟函数
- `@/utils/day.js` - `fromNow` 相对时间格式化
- `@/enums/email-enum.js` - `EmailUnreadEnum` 邮件状态枚举

<!-- MANUAL: -->
