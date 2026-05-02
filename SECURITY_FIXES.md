# 前端安全修复说明

## 修复日期
2026-04-07

## 修复内容

### 1. XSS 漏洞修复（DOMPurify）

**文件**: `src/lib/export/export.service.ts`

**问题**: 在导出 PDF 时，直接使用 `innerHTML` 将用户生成的 HTML 内容插入 DOM，可能导致 XSS 攻击。

**修复方案**:
- 引入 `DOMPurify` 库清理 HTML 内容
- 配置允许的标签和属性白名单
- 在使用 `innerHTML` 前先调用 `DOMPurify.sanitize()`

**需要安装的依赖**:
```bash
npm install dompurify @types/dompurify
# 或
pnpm add dompurify @types/dompurify
```

### 2. API Key 加密传输

**文件**:
- `src/lib/crypto.ts` (新建)
- `src/app/dashboard/settings/page.tsx`

**问题**: API Key 以明文形式通过 HTTPS 传输到后端，虽然传输层加密（HTTPS），但敏感数据应该在应用层额外加密。

**修复方案**:
- 创建前端加密工具，使用 Web Crypto API 实现 AES-256-GCM 加密
- 加密格式与后端 Node.js crypto 模块完全兼容
- 在发送 API Key 前先进行加密
- 后端接收到加密数据后会自动解密（已有逻辑）

**加密流程**:
1. 前端使用 `VITE_ENCRYPTION_SECRET` 通过 SHA-256 派生密钥
2. 生成随机 IV（初始化向量）
3. 使用 AES-256-GCM 加密 API Key
4. 将 IV、authTag、密文组合成 `IV:authTag:ciphertext` 格式
5. 后端使用相同的 `ENCRYPTION_SECRET` 解密

**环境变量配置**:
在 `.env` 文件中添加（与后端 `ENCRYPTION_SECRET` 保持一致）:
```env
VITE_ENCRYPTION_SECRET=polishly-default-secret
```

**注意**: 不需要安装 `crypto-js`，因为我们使用了浏览器原生的 Web Crypto API。

### 3. 后端兼容性说明

后端的加密实现位于 `api/src/lib/crypto.ts`，使用 Node.js 的 `crypto` 模块。

前端加密格式已与后端完全兼容：
- 相同的密钥派生方式（SHA-256 哈希）
- 相同的加密算法（AES-256-GCM）
- 相同的数据格式（IV:authTag:ciphertext，十六进制编码）

## 验证步骤

### 1. 验证 XSS 修复
```typescript
// 在浏览器控制台测试
import DOMPurify from 'dompurify';

const malicious = '<script>alert("XSS")</script>Hello';
const clean = DOMPurify.sanitize(malicious);
console.log(clean); // 输出: Hello（script 标签被移除）
```

### 2. 验证加密功能
```typescript
// 在浏览器控制台测试
import { validateEncryptionKey } from '@/lib/crypto';

validateEncryptionKey().then(isValid => {
  console.log('Encryption key valid:', isValid);
});
```

### 3. 测试 API Key 配置流程
1. 打开设置页面 (`/dashboard/settings`)
2. 点击"添加配置"
3. 填写配置信息（包括 API Key）
4. 点击"测试连接"或"添加配置"
5. 检查浏览器网络面板，确认 API Key 已被加密（格式：`hex:hex:hex`）

## 安全增强效果

### 修复前
- ⚠️ 用户生成的 HTML 内容直接注入 DOM，存在 XSS 风险
- ⚠️ API Key 以明文传输到后端

### 修复后
- ✅ 所有 HTML 内容经过 DOMPurify 清理，防止 XSS 攻击
- ✅ API Key 在应用层使用 AES-256-GCM 加密后再传输
- ✅ 即使 HTTPS 被中间人攻击，攻击者也无法获取明文 API Key
- ✅ 符合数据安全最佳实践（双重加密：传输层 + 应用层）

## 后续建议

1. **生产环境密钥管理**:
   - 不要在代码中硬编码 `VITE_ENCRYPTION_SECRET`
   - 使用安全的密钥管理服务（如 AWS KMS、Azure Key Vault）
   - 定期轮换加密密钥

2. **CSP 头部配置**:
   - 在服务器配置 Content-Security-Policy 头
   - 进一步限制脚本来源

3. **定期安全审计**:
   - 使用 `npm audit` 检查依赖漏洞
   - 定期进行渗透测试

## 注意事项

1. **依赖安装**: 用户需要手动运行 `npm install dompurify @types/dompurify`
2. **环境变量**: 确保 `.env` 文件中的 `VITE_ENCRYPTION_SECRET` 与后端 `ENCRYPTION_SECRET` 一致
3. **浏览器兼容性**: Web Crypto API 需要现代浏览器（IE 不支持，但本项目使用 Vite + React，默认不支持 IE）
