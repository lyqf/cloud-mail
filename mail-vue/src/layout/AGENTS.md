<!-- Parent: ../AGENTS.md -->
<!-- Generated: 2026-03-18 -->

# layout

## Purpose
应用布局组件，定义页面结构。包含顶部导航栏、侧边栏、主内容区域、账户切换面板和邮件写入界面。

## Architecture

```
Layout (index.vue)
  ├── Aside (aside/) - 侧边栏导航
  ├── Header (header/) - 顶部导航栏
  ├── Main (main/) - 主内容区域
  │   ├── Account (account/) - 账户切换面板
  │   └── router-view - 页面视图
  └── Write (write/) - 邮件写入界面
```

## Key Files

| File | Purpose |
|------|---------|
| `index.vue` | 主布局容器，组织所有子组件，处理响应式侧边栏和移动端适配 |
| `aside/index.vue` | 侧边栏导航菜单，包含邮箱菜单项和管理后台菜单 |
| `header/index.vue` | 顶部导航栏，包含面包屑、写邮件按钮、主题切换、通知和用户下拉菜单 |
| `main/index.vue` | 主内容区域容器，包含账户切换面板和 router-view |
| `account/index.vue` | 账户切换面板，支持多账户选择和管理 |
| `write/index.vue` | 邮件写入界面，支持邮件编辑、草稿、发送等功能 |

## Subdirectories

| Directory | Purpose |
|-----------|---------|
| `aside/` | 侧边栏导航组件，提供菜单项路由和权限控制 |
| `header/` | 顶部导航栏组件，包含工具栏和用户信息 |
| `main/` | 主内容区域容器，管理账户面板和路由视图显示 |
| `account/` | 账户切换面板，多账户管理界面 |
| `write/` | 邮件编辑界面，包含编辑器、草稿箱、发送逻辑 |

## Component Behavior

### index.vue (Main Layout)
- **响应式设计**：根据窗口宽度（1024px 断点）切换侧边栏显示方式
- **移动端适配**：小屏幕时侧边栏以浮层形式展示，点击遮罩层关闭
- **状态管理**：通过 Pinia 的 `uiStore` 控制侧边栏和账户面板显示
- **事件监听**：监听窗口 resize 事件，动态调整响应式状态

### aside/index.vue (Sidebar)
- **菜单导航**：email、send、draft、star、setting（用户菜单）
- **管理菜单**：analysis、user、all-email、role、reg-key、sys-setting（管理员菜单，权限控制）
- **权限控制**：使用 `v-perm` 指令显示/隐藏菜单项
- **活跃指示**：根据当前路由高亮对应菜单项

### header/index.vue (Header)
- **面包屑导航**：显示当前页面标题
- **汉堡菜单**：切换侧边栏显示/隐藏
- **写邮件按钮**：打开邮件写入界面（权限控制 `email:send`）
- **工具栏**：
  - 主题切换（深色/浅色模式）
  - 通知中心
  - 用户信息下拉菜单
- **用户菜单**：显示用户名、邮箱、角色、发送配额、账户数量、登出按钮

### main/index.vue (Main Content)
- **账户切换面板**：显示多账户选择界面（条件：`accountShow && hasPerm('account:query')`）
- **遮罩层**：点击关闭账户面板
- **router-view**：渲染当前路由对应的页面视图
- **KeepAlive 缓存**：缓存已访问的页面组件，保持状态

### account/index.vue (Account Panel)
- **账户列表**：显示所有可用账户
- **账户切换**：点击切换当前账户
- **账户管理**：添加、编辑、删除账户（需要权限）

### write/index.vue (Email Writer)
- **邮件编辑器**：富文本编辑邮件内容
- **收件人管理**：添加、删除收件人
- **草稿保存**：自动或手动保存草稿
- **邮件发送**：发送邮件（需要权限 `email:send`）
- **附件上传**：支持文件附件

## State Management

### UI Store (useUiStore)
```javascript
{
  asideShow: boolean,      // 侧边栏是否显示
  accountShow: boolean,    // 账户切换面板是否显示
  dark: boolean,           // 深色模式
  writerRef: ref,          // 邮件写入组件的引用
  changeNotice: number,    // 通知变化触发器
  changePreview: number,   // 预览变化触发器
  previewData: object      // 预览数据
}
```

### Setting Store (useSettingStore)
```javascript
{
  settings: {
    title: string,         // 应用标题
    manyEmail: number,     // 多账户限制
    addEmail: boolean,     // 是否允许添加账户
    notice: object,        // 通知配置
    noticeWidth: number,   // 通知宽度
    noticeTitle: string,   // 通知标题
    noticeContent: string, // 通知内容
    noticeType: string,    // 通知类型
    noticeDuration: number,// 通知持续时间
    noticePosition: string,// 通知位置
    noticeOffset: number   // 通知偏移
  }
}
```

## Key Dependencies

### Internal
- `@/store/ui.js` - UI 状态管理
- `@/store/setting.js` - 应用设置管理
- `@/store/user.js` - 用户信息管理
- `@/request/login.js` - 登出 API
- `@/perm/perm.js` - 权限检查函数
- `@/components/hamburger/` - 汉堡菜单组件

### External
- `element-plus` - UI 组件库（el-container, el-aside, el-header, el-main, el-menu, el-dropdown, el-button, el-tag）
- `vue-router` - 路由管理
- `@iconify/vue` - 图标库
- `vue` - Composition API (ref, computed, watch, onMounted, onBeforeUnmount)

## Responsive Design

| Screen Size | Behavior |
|-------------|----------|
| > 1024px | 侧边栏始终显示，汉堡菜单隐藏 |
| ≤ 1024px | 侧边栏默认隐藏，汉堡菜单显示，点击显示侧边栏浮层 |

## For AI Agents

### Working In This Directory
- 修改布局结构时，确保保留所有子组件的引入和挂载
- 调整响应式断点时，同时修改 `index.vue` 和样式中的 1024px 值
- 添加新的顶部工具栏按钮在 `header/` 中实现
- 添加新的侧边栏菜单项在 `aside/` 中实现
- 邮件编辑功能集中在 `write/` 中实现

### Common Patterns

**在 Header 中添加新工具栏按钮：**
```javascript
<div class="icon-item" @click="handleNewAction">
  <Icon icon="icon-name" />
</div>
```

**在 Aside 中添加新菜单项：**
```javascript
<el-menu-item
  @click="router.push({name: 'new-page'})"
  v-perm="'permission:name'"
  :class="route.meta.name === 'new-page' ? 'choose-item' : ''">
  <Icon icon="icon-name" width="20" height="20" />
  <span class="menu-name">{{ $t('label') }}</span>
</el-menu-item>
```

**在 Main 中添加新的 KeepAlive 缓存：**
```javascript
<keep-alive :include="['email', 'new-view', ...]">
  <component :is="Component" :key="route.name"/>
</keep-alive>
```

### Testing Requirements
- 修改后运行 `pnpm dev` 查看布局效果
- 检查浏览器窗口大小改变时，响应式行为是否正确
- 验证权限控制是否生效（某些菜单项应该隐藏）
- 确保移动端侧边栏可以正常打开和关闭

### Common Issues
- **侧边栏显示异常**：检查 `uiStore.asideShow` 状态和窗口宽度判断逻辑
- **菜单项不显示**：检查权限指令 `v-perm` 和用户权限配置
- **样式污染**：检查 scoped 样式是否正确，避免全局样式影响
- **响应式不工作**：检查 resize 事件监听是否正确移除，避免内存泄漏

<!-- MANUAL: -->
