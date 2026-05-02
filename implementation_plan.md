# Polishly 增强错误处理实施计划

**版本**: 1.0  
**创建日期**: 2026-04-04  
**项目**: Polishly 专利智能生成平台  
**范围**: 前后端统一错误处理系统

---

## 1. 执行摘要

当前项目存在不一致的错误处理模式，包括：
- 使用 `console.warn` 记录严重错误
- 缺少统一的错误类型定义
- 错误信息不够详细，缺乏错误码
- 没有错误恢复机制和友好的用户提示

本计划旨在构建统一的错误处理系统，提升系统健壮性和用户体验。

---

## 2. 架构设计

### 2.1 前端架构

```
src/lib/errors/
├── types.ts              # 错误类型定义和错误码枚举
├── error-boundary.tsx    # React 错误边界组件
├── error-handler.ts      # 全局错误处理器
├── retry-utils.ts        # 重试机制工具
└── index.ts              # 导出

src/components/ui/
├── toast.tsx             # Toast 通知组件
└── error-display.tsx     # 错误展示组件
```

### 2.2 后端架构

```
api/src/common/
├── errors/
│   ├── app-error.ts      # 自定义错误基类
│   ├── error-codes.ts    # 错误码枚举
│   ├── error-response.ts # 统一错误响应格式
│   └── index.ts
└── middleware/
    └── error-handler.ts  # 全局错误处理中间件
```

---

## 3. 类型定义

### 3.1 前端错误类型 (src/lib/errors/types.ts)

```typescript
// 错误码枚举
export enum ErrorCode {
  // 通用错误 (1000-1999)
  UNKNOWN_ERROR = 'E1000',
  NETWORK_ERROR = 'E1001',
  VALIDATION_ERROR = 'E1002',
  TIMEOUT_ERROR = 'E1003',
  
  // 认证错误 (2000-2999)
  AUTH_REQUIRED = 'E2000',
  AUTH_EXPIRED = 'E2001',
  
  // 专利生成错误 (3000-3999)
  GENERATION_FAILED = 'E3000',
  SECTION_GENERATION_FAILED = 'E3001',
  OUTLINE_GENERATION_FAILED = 'E3002',
  CONTENT_VALIDATION_FAILED = 'E3003',
  
  // AI Provider 错误 (4000-4999)
  PROVIDER_NOT_CONFIGURED = 'E4000',
  PROVIDER_API_ERROR = 'E4001',
  PROVIDER_RATE_LIMIT = 'E4002',
  PROVIDER_QUOTA_EXCEEDED = 'E4003',
  
  // 导出错误 (5000-5999)
  EXPORT_FAILED = 'E5000',
  EXPORT_FORMAT_UNSUPPORTED = 'E5001',
}

// 错误级别
export enum ErrorLevel {
  INFO = 'info',
  WARNING = 'warning',
  ERROR = 'error',
  CRITICAL = 'critical',
}

// 应用错误接口
export interface AppError extends Error {
  code: ErrorCode;
  level: ErrorLevel;
  details?: Record<string, unknown>;
  timestamp: Date;
  retryable: boolean;
}

// API 错误响应
export interface ApiErrorResponse {
  error: {
    code: string;
    message: string;
    details?: Record<string, unknown>;
    timestamp: string;
  };
}
```

### 3.2 后端错误类型 (api/src/common/errors/app-error.ts)

```typescript
export enum HttpStatusCode {
  BAD_REQUEST = 400,
  UNAUTHORIZED = 401,
  FORBIDDEN = 403,
  NOT_FOUND = 404,
  TIMEOUT = 408,
  CONFLICT = 409,
  UNPROCESSABLE_ENTITY = 422,
  TOO_MANY_REQUESTS = 429,
  INTERNAL_SERVER_ERROR = 500,
  SERVICE_UNAVAILABLE = 503,
}

export enum ErrorCode {
  // 通用错误
  UNKNOWN_ERROR = 'E1000',
  VALIDATION_ERROR = 'E1001',
  INTERNAL_ERROR = 'E1002',
  
  // 专利相关
  PATENT_NOT_FOUND = 'E3001',
  PATENT_GENERATION_FAILED = 'E3002',
  CONTENT_TOO_LONG = 'E3003',
  
  // Provider 相关
  PROVIDER_NOT_FOUND = 'E4001',
  PROVIDER_CONFIG_INVALID = 'E4002',
  PROVIDER_API_FAILED = 'E4003',
  PROVIDER_RATE_LIMITED = 'E4004',
}

export class AppError extends Error {
  public readonly code: string;
  public readonly statusCode: HttpStatusCode;
  public readonly details?: Record<string, unknown>;
  public readonly isOperational: boolean;

  constructor(
    message: string,
    code: string,
    statusCode: HttpStatusCode,
    details?: Record<string, unknown>,
    isOperational = true
  ) {
    super(message);
    this.name = 'AppError';
    this.code = code;
    this.statusCode = statusCode;
    this.details = details;
    this.isOperational = isOperational;
    
    Object.setPrototypeOf(this, AppError.prototype);
    Error.captureStackTrace(this, this.constructor);
  }
}
```

---

## 4. 文件清单

### 4.1 新建文件

| 文件路径 | 描述 | 优先级 |
|---------|------|--------|
| `src/lib/errors/types.ts` | 错误类型定义 | P0 |
| `src/lib/errors/error-boundary.tsx` | React Error Boundary | P0 |
| `src/lib/errors/error-handler.ts` | 全局错误处理器 | P0 |
| `src/lib/errors/retry-utils.ts` | 重试工具函数 | P1 |
| `src/components/ui/toast.tsx` | Toast 通知组件 | P0 |
| `api/src/common/errors/app-error.ts` | 自定义错误类 | P0 |
| `api/src/common/errors/error-codes.ts` | 错误码枚举 | P0 |
| `api/src/common/errors/error-response.ts` | 统一响应格式 | P0 |
| `api/src/common/middleware/error-handler.ts` | 错误处理中间件 | P0 |

### 4.2 修改文件

| 文件路径 | 修改内容 | 优先级 |
|---------|---------|--------|
| `src/lib/api-client.ts` | 增强错误处理，添加统一错误解析 | P0 |
| `src/lib/ai-service.ts` | 使用新的错误类型 | P0 |
| `src/app/dashboard/page.tsx` | 移除 console.warn，添加 Toast 提示 | P1 |
| `src/app/dashboard/patents/new/page.tsx` | 添加重试机制和友好错误提示 | P1 |
| `src/App.tsx` | 添加 ErrorBoundary 包装 | P0 |
| `api/src/index.ts` | 使用新的错误处理中间件 | P0 |
| `api/src/modules/generation/generation.service.ts` | 使用 AppError | P1 |

---

## 5. 核心函数和类

### 5.1 前端核心

#### ErrorBoundary 组件
```typescript
// 位置: src/lib/errors/error-boundary.tsx
interface Props {
  children: React.ReactNode;
  fallback?: React.ReactNode;
  onError?: (error: AppError, errorInfo: React.ErrorInfo) => void;
}

// 功能:
// - 捕获子组件渲染错误
// - 记录错误到控制台
// - 展示友好错误 UI
// - 支持自定义 fallback
```

#### 错误处理 Hook
```typescript
// 位置: src/lib/errors/use-error-handler.ts (新建)
export function useErrorHandler() {
  function handleError(error: unknown): AppError;
  function handleApiError(response: Response): Promise<never>;
  function reportError(error: AppError): void;
}

// 功能:
// - 统一错误转换
// - API 错误解析
// - 错误上报
```

#### 重试工具
```typescript
// 位置: src/lib/errors/retry-utils.ts
interface RetryOptions {
  maxAttempts?: number;      // 默认 3
  delayMs?: number;          // 默认 1000
  backoffMultiplier?: number; // 默认 2
  retryableCodes?: string[]; // 可重试的错误码
}

export async function withRetry<T>(
  fn: () => Promise<T>,
  options?: RetryOptions
): Promise<T>;

// 功能:
// - 指数退避重试
// - 可配置重试条件
// - 返回详细错误信息
```

### 5.2 后端核心

#### AppError 类
```typescript
// 位置: api/src/common/errors/app-error.ts

// 静态工厂方法
AppError.badRequest(message: string, details?: object)
AppError.notFound(resource: string)
AppError.unauthorized(message?: string)
AppError.internal(message?: string)
AppError.serviceUnavailable(message?: string)

// 方法
error.toJSON() // 转换为统一响应格式
error.toLog()  // 转换为日志格式
```

#### 错误处理中间件
```typescript
// 位置: api/src/common/middleware/error-handler.ts

export function errorHandler(
  error: Error,
  request: FastifyRequest,
  reply: FastifyReply
): FastifyReply;

// 功能:
// - 区分 AppError 和普通 Error
// - 统一错误响应格式
// - 错误日志记录
// - 生产环境隐藏内部细节
```

---

## 6. 依赖关系

### 6.1 前端新增依赖

```json
{
  "dependencies": {
    "sonner": "^1.4.0"
  }
}
```

### 6.2 后端依赖

无新增依赖，使用 Fastify 内置功能。

---

## 7. API 错误响应格式

### 7.1 统一错误响应结构

```typescript
interface ErrorResponse {
  error: {
    code: string;           // 错误码，如 "E1001"
    message: string;        // 用户友好的消息
    details?: {             // 可选的详细信息
      field?: string;
      value?: unknown;
      suggestion?: string;
    };
    timestamp: string;      // ISO 8601 格式时间
    requestId?: string;     // 请求追踪 ID
  };
}
```

### 7.2 响应示例

```json
{
  "error": {
    "code": "E3002",
    "message": "专利内容生成失败",
    "details": {
      "section": "abstract",
      "suggestion": "请稍后重试或减少内容长度"
    },
    "timestamp": "2026-04-04T18:00:00.000Z",
    "requestId": "req_abc123"
  }
}
```

---

## 8. 测试策略

### 8.1 单元测试

```typescript
// tests/unit/errors/error-handler.test.ts
describe('ErrorHandler', () => {
  it('should convert unknown error to AppError');
  it('should preserve error code from ApiErrorResponse');
  it('should add timestamp to new errors');
});

// tests/unit/errors/retry-utils.test.ts
describe('withRetry', () => {
  it('should retry on transient failures');
  it('should not retry on non-retryable errors');
  it('should respect maxAttempts limit');
  it('should implement exponential backoff');
});
```

### 8.2 集成测试

```typescript
// tests/e2e/error-handling.spec.ts
describe('Error Handling E2E', () => {
  it('should display user-friendly error on API failure');
  it('should show retry option on generation failure');
  it('should log errors with correct severity');
});
```

---

## 9. 实施顺序

### Phase 1: 基础设施 (优先级 P0)

1. **创建错误类型定义** 
   - `src/lib/errors/types.ts`
   - `api/src/common/errors/error-codes.ts`

2. **创建自定义错误类**
   - `api/src/common/errors/app-error.ts`
   - `api/src/common/errors/error-response.ts`

3. **创建 Error Boundary**
   - `src/lib/errors/error-boundary.tsx`
   - 更新 `src/App.tsx`

### Phase 2: UI 组件 (优先级 P0)

4. **创建 Toast 通知组件**
   - 安装 `sonner` 依赖
   - `src/components/ui/toast.tsx`

5. **更新 API 客户端**
   - `src/lib/api-client.ts`
   - 添加错误解析和统一处理

### Phase 3: 服务层 (优先级 P1)

6. **创建错误处理器**
   - `src/lib/errors/error-handler.ts`
   - `src/lib/errors/retry-utils.ts`

7. **更新 AI 服务**
   - `src/lib/ai-service.ts`
   - 使用新的错误类型

### Phase 4: 后端中间件 (优先级 P0)

8. **创建错误处理中间件**
   - `api/src/common/middleware/error-handler.ts`
   - 更新 `api/src/index.ts`

9. **更新业务服务**
   - `api/src/modules/generation/generation.service.ts`

### Phase 5: 页面集成 (优先级 P1)

10. **更新仪表板页面**
    - `src/app/dashboard/page.tsx`
    - 移除 `console.warn`，使用 Toast

11. **更新专利生成页面**
    - `src/app/dashboard/patents/new/page.tsx`
    - 添加重试机制

### Phase 6: 测试 (优先级 P2)

12. **编写单元测试**
13. **编写 E2E 测试**

---

## 10. 向后兼容性

- 错误码格式保持一致：`E{category}{number}`
- API 响应添加 `error.code` 字段，现有 `message` 字段保留
- 错误边界不会影响正常功能，仅在错误时生效

---

## 11. 监控和日志

### 11.1 日志级别

| 场景 | 级别 | 示例 |
|------|------|------|
| 网络错误 | error | 连接超时、DNS 失败 |
| API 错误 | error | 返回 4xx/5xx |
| 验证错误 | warning | 表单验证失败 |
| 生成失败 | error | AI 内容生成失败 |
| 用户操作失败 | info | 导出失败 |

### 11.2 监控指标

- 错误率：每分钟错误数 / 总请求数
- 错误分布：错误码占比
- 用户影响：错误导致的页面崩溃次数

---

## 12. 风险评估

| 风险 | 影响 | 缓解措施 |
|------|------|---------|
| 错误边界影响性能 | 中 | 使用 React.memo 优化 |
| 依赖版本冲突 | 低 | 使用已有生态库 sonner |
| API 响应格式变更 | 中 | 保持 message 字段兼容 |
| 测试覆盖不足 | 高 | 优先编写核心测试 |

---

## 13. 预期成果

实施完成后，系统将具备：

✅ 统一的错误类型和错误码系统  
✅ 全局错误边界，防止页面崩溃  
✅ 友好的错误提示 UI  
✅ 智能重试机制  
✅ 完整的错误日志  
✅ 可追踪的请求 ID  
✅ 详细的错误分析数据  
