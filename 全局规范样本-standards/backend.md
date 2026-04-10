# 后端编码规范（通用模板）

> 项目启动时复制此文件到 `docs/config/STANDARDS.md`，按项目实际情况增删。
> 通用规则在此维护；项目特有规则写在项目 STANDARDS.md 末尾，不要改这里。

---

## 分层规范（强制）

- Controller 只做路由接收和参数传递，**禁止在 Controller 层写业务逻辑判断**
- 业务逻辑全部在 Service 层
- **禁止 Controller 直接操作数据库**，必须通过 TypeORM Repository 或 Service
- 禁止在 Service 层直接操作 response 对象，统一由拦截器处理

## 响应格式规范（强制）

- Controller 直接 return 数据，**禁止手动包装 `{ code, msg, data }`**
- 所有成功响应由 `SuccessInterceptor` 统一格式化
- 所有异常由 `ExceptionFilter` 统一处理，业务代码不直接操作 response 对象

## 入参验证（强制）

- 所有 DTO **必须有完整的 class-validator 装饰器**
- 必填字段必须有 `@IsNotEmpty()` 或 `@IsDefined()`
- 字段类型必须有明确声明：`@IsString()`、`@IsNumber()`、`@IsBoolean()` 等
- 禁止用隐式类型转换替代显式验证
- 外部 API 返回值（第三方 SDK）必须检查关键字段是否为 null/undefined，不能直接解构

## 安全规范（强制）

- 禁止硬编码密钥、密码、API Key，统一通过环境变量注入
- 禁止明文存储任何凭证类数据，加密方案在项目 STANDARDS.md 中指定
- 响应数据中禁止出现 `password`、`secret`、`token` 等敏感字段明文
- 需要鉴权的接口必须有 `@UseGuards`，不得遗漏
- 附件/文件下载接口必须校验资源归属用户，防止越权访问

## 日志规范

- 禁止在日志中打印用户隐私数据（邮件正文、密码、token 等）
- 禁止在日志中记录用户输入内容（尤其是 AI 相关日志）
- 错误日志必须记录具体原因，不能只 `catch` 不记录

## 数据库规范

- 禁止 N+1 查询：关联数据用 `relations` 或 `QueryBuilder` 一次性加载
- 有唯一性要求的字段在 Entity 层加 `unique: true`，同时确认数据库层 UNIQUE KEY 实际生效
- upsert 操作需验证唯一约束实际有效

## 代码整洁

- 禁止提交含 `// TODO`、`// FIXME` 的未完成代码
- 禁止遗留 `console.log`（用 NestJS Logger 替代）
- 禁止提交注释掉的废弃代码块
- 禁止未使用的 import

## 测试规范

### Jest mock 类型转换

TypeScript 不允许直接 `(X as jest.Mock)`，需双重转换：

```typescript
(ImapFlow as unknown as jest.Mock).mockImplementation(() => ({ ... }));
```

### jest.mock('fs') 注意事项

直接 mock `fs` 会破坏 TypeORM 的 glob 初始化（`path-scurry` 内部调用 `fs.statSync`）。
**正确写法**：用 `jest.requireActual` 保留真实 fs，只 override 需要的方法：

```typescript
jest.mock('fs', () => {
  const actual = jest.requireActual<typeof import('fs')>('fs');
  return {
    ...actual,
    promises: {
      ...actual.promises,
      mkdir:     jest.fn().mockResolvedValue(undefined),
      writeFile: jest.fn().mockResolvedValue(undefined),
    },
  };
});
```

### 文件路径安全

处理用户提供/外部来源的文件名，必须先 `path.basename()` 剥离目录成分，再用正则过滤：

```typescript
const safeName = path.basename(rawName).replace(/[^a-zA-Z0-9._-]/g, '_') || `file-${Date.now()}`;
```

单独使用正则（如 `replace(/\.\./g, '')`）无法防止 `../../../etc/passwd` 形式的路径穿越。
