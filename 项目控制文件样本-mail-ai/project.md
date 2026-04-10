# 项目全景 · mail-ai

## 产品层

### 产品目标

为内部团队（≤50 人）提供 AI 驱动的独立邮件客户端，通过智能分类和信息提取帮助团队成员快速处理每日 30～100 封工作邮件，并与飞书任务打通，减少手动整理时间。

### 目标用户

内部团队成员，每日处理大量工作邮件，需要快速筛选重要邮件并跟进待办事项。

### 核心功能边界

- 飞书 OAuth 是唯一登录方式，不提供用户名密码登录
- 每用户最多绑定 3 个邮箱账户，超出返回 409
- 邮件收发：IMAP 收取 + SMTP 发送
- AI 处理：分类、摘要、待办提取、关键日期提取
- 飞书集成：任务同步、消息分享

---

## 技术层

### 技术选型

- 前端：Vue 3.5 + TypeScript 5.9 + Vite 5.4 + Element Plus 2.13 + Pinia 3.0 + Vue Router 4.6 + SCSS
- 后端：NestJS 10.3 + TypeScript 5.7 + TypeORM 0.3 + MySQL2（无 @nestjs/schedule）
- 数据库：MySQL 8.0
- 缓存 / 任务队列：Redis 7.x + Bull
- 邮件协议：imapflow（IMAP）+ nodemailer（SMTP）
- AI：Claude API（主）/ OpenAI API（备）
- 飞书：直接调用 open-api（不用 SDK，SDK 会吞错误）
- 部署：Nginx + PM2，服务器自部署
- 客户端：飞书 H5（webview）+ 响应式 Web

### 目录结构

```
mail-ai/
├── frontend/src/
│   ├── api/          # 接口封装，按模块分文件
│   ├── components/   # 公共组件
│   ├── views/        # 页面组件
│   ├── stores/       # Pinia 状态管理
│   ├── router/       # 路由配置
│   ├── composables/  # 组合式函数
│   ├── utils/        # 工具函数
│   ├── styles/       # SCSS 变量和全局样式
│   └── types/        # TypeScript 类型定义
└── backend/src/
    ├── modules/      # 业务模块（auth、mail-accounts、emails、todos 等）
    ├── common/       # 公共装饰器、过滤器、守卫
    ├── config/       # 配置模块（数据库、Redis、AI API）
    └── queue/        # Bull 队列处理器
```

### 技术约束

- IMAP 密码和飞书 token 必须 AES-256 加密存储，密钥通过环境变量注入
- AI 处理队列并发数设为 3，不得随意调高
- 新邮件 AI 处理任务优先级高于历史邮件批量任务（priority=1 vs priority=10）
- 附件存本地磁盘，路径字段命名 `storage_path`，不存 BLOB
- JWT payload 使用 `sub` 字段存储用户 ID，禁止用 `req.user.id`

### AI 邮件处理流程（四期当前实现）

```
新邮件到达
  → Bull 队列（并发 3，新邮件 priority=1，历史 priority=10）
  → AI 打标（mail_type / priority / action_required）
  → 写 inbox_card_type（priority=high → important，否则 → normal）
```

### 多账户上下文机制

所有涉及邮件数据的接口支持可选 Header `X-Account-Id: {accountId}`：
- 传入时：按指定账户筛选
- 不传时：默认使用用户第一个账户（向后兼容）
