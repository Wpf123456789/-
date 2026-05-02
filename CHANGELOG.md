# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-04-08

### Added

#### 设计系统
- **设计令牌系统**: 创建完整的设计变量系统（颜色、间距、圆角、阴影、排版）
- **CSS 变量驱动**: 支持亮色和暗色模式，基于 4px 基准的 8px 网格系统
- **全局样式增强**: 更新 globals.css，添加基础重置和丰富动画

#### UI 组件库
- **Button 组件**: 5种变体（primary/secondary/outline/ghost/danger）、3种尺寸、loading状态、图标支持、微交互动画
- **Input 组件**: 3种变体（default/error/success）、3种尺寸、浮动标签、验证状态、图标支持
- **Card 组件**: 4种变体（default/elevated/outlined/bordered）、hover效果、clickable交互、子组件系统
- **Spinner 组件**: 加载指示器（多尺寸、多颜色）
- **Skeleton 组件**: 骨架屏（多形状、批量生成、shimmer动画）
- **EmptyState 组件**: 空状态展示（支持操作按钮、多尺寸）
- **LoadingOverlay 组件**: 全屏/局部加载遮罩（3种变体）
- **PageTransition 组件**: 页面过渡动画（fade/slide/scale）
- **Container 组件**: 响应式容器组件（5种尺寸）

#### 移动端优化
- **MobileNav 组件**: 移动端抽屉导航、平滑动画
- **HamburgerButton 组件**: 汉堡菜单按钮、3条线变形动画
- **触摸优化**: 最小触摸目标 44x44px、防双击缩放、触摸反馈
- **响应式导航**: 水平滚动步骤导航、简化标签

#### 页面优化
- **首页**: 使用新组件系统、渐变背景、现代化设计、响应式布局
- **仪表板**: 统计卡片、骨架屏加载、空状态优化、响应式网格
- **设置页面**: Card组织、表单优化、Toast通知、响应式布局
- **专利生成向导**: 步骤导航、SVG图标、加载反馈、表单验证

### Changed

- **微交互优化**: 按钮、输入框、卡片的悬停/焦点/点击效果
- **动画系统**: 所有动画使用 CSS transitions/keyframes，GPU 加速
- **响应式布局**: 所有页面从 320px 到 1440px+ 完美适配
- **性能优化**: will-change 属性、transform 替代 top/left

### Fixed

- **无障碍支持**: 所有组件添加完整 ARIA 属性
- **焦点管理**: focus-visible 优化键盘导航体验
- **颜色对比度**: 符合 WCAG AA 标准
- **布局抖动**: 消除加载时的布局偏移

## [1.0.1] - 2026-03-30

### Security

- **认证系统移除**: 移除 JWT 认证系统，改为前端 hash 路由隔离，简化架构
- **API Key 加密**: 实现 AES-256-GCM 加密存储（前端 Web Crypto API + 后端 Node.js crypto）
- **Rate Limiting**: 添加全局速率限制（100 请求/分钟）和认证路由限制（20 请求/分钟）
- **CSP Headers**: 添加 Content-Security-Policy 头部，防止注入攻击
- **XSS 防护**: 集成 DOMPurify 清理用户生成的 HTML 内容
- **CORS 配置**: 可配置的跨域白名单

### Fixed

- **输入验证**: 添加 Zod schema 验证所有 API 输入
- **错误处理**: 统一错误处理逻辑和响应格式
- **API 规范**: RESTful API 规范化（DELETE 返回 204）

### Removed

- **认证模块**: 删除 `/api/v1/auth/*` 路由和相关代码
- **用户模块**: 删除 User 和 RefreshToken 数据模型
- **登录/注册页面**: 删除前端登录注册页面

详细说明请参阅 [SECURITY_FIXES.md](./SECURITY_FIXES.md)

## [1.0.0] - 2026-03-30

### Changed

- **Frontend Build**: Removed `"use client"` directive from 3 files (settings/page.tsx, polish-panel.tsx, wizard-steps.tsx) - these are Next.js specific and not compatible with Vite
- **README.md**: Updated technology stack description - removed "Radix UI primitives" as it was never used
- **README.md**: Updated project structure to reflect current codebase (removed login/register pages and auth/users modules)

### Removed

- **Frontend Dead Code**: Deleted 7 unused files:
  - `src/app/login/page.tsx`
  - `src/app/register/page.tsx`
  - `src/lib/errors.ts`
  - `src/lib/id.ts`
  - `src/lib/time.ts`
  - `src/lib/prompts.ts`
  - `src/types/user.ts`
- **Backend Auth Modules**: Removed obsolete authentication code:
  - `api/src/modules/auth/auth.routes.ts`
  - `api/src/modules/auth/auth.service.ts`
  - `api/src/modules/auth/dto/auth.dto.ts`
  - `api/src/modules/users/users.routes.ts`
- **Archived Documentation**: Moved outdated chat-related rules to `.trae/documents/`:
  - `chat-flow.md` → `archived-chat-flow.md`
  - `frontend-chat-mvp.md` → `archived-frontend-chat-mvp.md`

### Fixed

- **Build Warnings**: Resolved Vite build warnings about incompatible `"use client"` directives

### Security

- **Dependency Cleanup**: Removed unused authentication dependencies from backend package.json

## [0.0.1] - Previous

- Initial project setup from text polishing tool to patent generation platform
