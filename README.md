

## 技术栈

### 前端
- **框架**: Vite 5 + React 18 + TypeScript 5.6
- **样式**: Tailwind CSS + CSS Variables (Design Tokens)
- **表单**: React Hook Form + Zod 验证
- **状态**: TanStack Query + Zustand
- **UI 组件**: 自定义组件库（Button、Input、Card、Spinner、Skeleton、EmptyState）
- **工具**: DOMPurify (XSS防护)、html2pdf.js (文档导出)
- **测试**: Vitest + Testing Library + Playwright
- **构建**: TypeScript Compiler + Vite

### 后端
- **框架**: Fastify + Node.js
- **数据库**: Prisma ORM + SQLite
- **AI**: 多协议 AI Provider（OpenAI/MiniMax/Anthropic 兼容）
- **安全**: AES-256-GCM 加密、Rate Limiting、CSP Headers
- **测试**: Vitest

## 环境要求

- Node.js 20.x
- pnpm 9.x

## 快速启动

```bash
pnpm install
pnpm dev:all
```

或分两个终端启动：

```bash
pnpm dev:api   # 后端，监听 3001
pnpm dev       # 前端，监听 5173
```

## 质量检查

```bash
pnpm typecheck    # TypeScript 类型检查
pnpm test         # 单元测试 + 集成测试
pnpm test:e2e     # E2E 测试（Playwright）
pnpm build        # 生产构建
```

## 环境变量

### 前端（`src/.env`）

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `VITE_API_BASE_URL` | `http://localhost:3001` | 后端 API 地址 |
| `VITE_API_TIMEOUT_MS` | `15000` | 请求超时（毫秒） |

### 后端（`api/.env`）

| 变量 | 必需 | 说明 |
|------|------|------|
| `DATABASE_URL` | 是 | SQLite 数据库路径，默认 `file:./dev.db` |
| `MINIMAX_API_KEY` | 是 | MiniMax API Key（环境变量兜底 Provider） |
| `MINIMAX_BASE_URL` | 是 | MiniMax API 地址 |
| `MINIMAX_MODEL` | 是 | MiniMax 模型名 |
| `PORT` | 否 | 服务端口，默认 `3001` |
| `UPSTREAM_TIMEOUT_MS` | 否 | 上游请求超时，默认 `8000` |
| `RATE_LIMIT_PER_MINUTE` | 否 | 限流，默认 `30` |
| `CORS_ORIGINS` | 否 | 跨域白名单，逗号分隔 |

**注意**：优先使用数据库中配置的 Provider（`/api/v1/providers`），环境变量仅作为全局兜底。

## 核心功能


### AI 模型配置
- 支持 OpenAI / MiniMax / Anthropic 三种协议
- 全局配置 + 用户级配置分层优先级
- API Key 加密存储（AES-256-GCM）
- 前端设置页面：`/dashboard/settings`
- 测试连接功能验证配置正确性

### 设计系统
- **设计令牌**: 统一的颜色、间距、圆角、阴影系统
- **响应式设计**: 完美适配移动端（320px）到桌面端（1440px+）
- **无障碍支持**: WCAG AA 标准、完整的 ARIA 属性
- **微交互**: 流畅的动画和过渡效果
- **加载状态**: 骨架屏、加载指示器、空状态处理

## API 路由

基础路径：`/api/v1`

**注意**: 所有 API 无需认证，依赖前端 hash 路由隔离。

| 方法 | 路径 | 说明 |
|------|------|------|
| `GET` | `/health` | 健康检查 |
| `POST` | `/generate` | 专利内容生成 |
| `GET` | `/providers` | 列出所有 Provider 配置 |
| `POST` | `/providers` | 创建 Provider 配置 |
| `PUT` | `/providers/:id` | 更新 Provider 配置 |
| `DELETE` | `/providers/:id` | 删除 Provider 配置 |
| `POST` | `/providers/:id/test` | 测试 Provider 连接 |
| `GET` | `/patents` | 专利列表 |
| `POST` | `/patents` | 创建专利 |
| `GET` | `/patents/:id` | 专利详情 |
| `PUT` | `/patents/:id` | 更新专利 |
| `DELETE` | `/patents/:id` | 删除专利 |
| `GET` | `/templates` | 模板列表 |
| `POST` | `/templates` | 创建模板 |
| `PUT` | `/templates/:id` | 更新模板 |
| `DELETE` | `/templates/:id` | 删除模板 |

## 项目结构

```
Polishly/
├── src/                        # 前端（Vite + React）
│   ├── app/
│   │   ├── dashboard/         # 仪表板页面
│   │   │   ├── patents/       # 专利管理（列表/新建/详情）
│   │   │   ├── settings/      # 模型配置页面
│   │   │   └── page.tsx       # 仪表板首页
│   │   └── page.tsx           # 首页
│   ├── components/
│   │   ├── generation/        # 生成向导组件
│   │   ├── layout/            # 布局组件（导航、页脚、移动端菜单）
│   │   └── ui/                # 基础 UI 组件
│   │       ├── button.tsx     # 按钮组件
│   │       ├── input.tsx      # 输入框组件
│   │       ├── card.tsx       # 卡片组件
│   │       ├── spinner.tsx    # 加载指示器
│   │       ├── skeleton.tsx   # 骨架屏
│   │       ├── empty-state.tsx # 空状态
│   │       ├── page-transition.tsx # 页面过渡
│   │       ├── loading-overlay.tsx # 加载遮罩
│   │       └── container.tsx  # 响应式容器
│   ├── lib/                   # 工具函数
│   │   ├── api-client.ts      # API 客户端
│   │   ├── export-service.ts  # 文档导出服务
│   │   ├── crypto.ts          # 加密工具
│   │   └── cn.ts              # className 合并
│   ├── stores/                # Zustand 状态管理
│   ├── types/                 # TypeScript 类型定义
│   ├── styles/
│   │   ├── design-tokens.css  # 设计令牌
│   │   └── globals.css        # 全局样式
│   └── index.css              # 样式入口
│
├── api/                        # 后端（Fastify + Prisma）
│   ├── src/
│   │   ├── modules/
│   │   │   ├── generation/    # AI 生成 + Provider 管理
│   │   │   ├── patents/       # 专利 CRUD
│   │   │   └── templates/     # 模板 CRUD
│   │   ├── common/            # 通用模块
│   │   │   ├── middleware/    # 中间件（速率限制、CSP）
│   │   │   └── errors/        # 错误处理
│   │   ├── config/            # 环境变量、数据库配置
│   │   ├── lib/               # 加密工具
│   │   └── index.ts           # 应用入口
│   ├── prisma/
│   │   ├── schema.prisma      # 数据模型（SQLite）
│   │   └── dev.db             # SQLite 数据库文件
│   └── .env                   # 后端环境变量
│
└── .trae/
    ├── rules/                 # 项目规则文件
    ├── skills/                # 自定义 Skill
    └── documents/             # 项目文档
```

## 安全特性

- **API Key 加密存储**: AES-256-GCM 加密，前端 Web Crypto API + 后端 Node.js crypto
- **XSS 防护**: DOMPurify 清理用户生成的 HTML 内容
- **Rate Limiting**: 全局 100 请求/分钟，认证路由 20 请求/分钟
- **CSP Headers**: Content-Security-Policy 防止注入攻击
- **CORS 配置**: 可配置的跨域白名单
- **输入验证**: Zod schema 验证所有 API 输入

详细安全修复说明请参阅 [SECURITY_FIXES.md](./SECURITY_FIXES.md)



迁移与种子：

```bash
cd api
pnpm prisma migrate dev     # 创建迁移
pnpm prisma db seed         # 填充种子数据
```
