# 复盘记录 · mail-ai

---

## 2026-04-04 · 技术助手 · TRD v1 阶段

### 技术决策质量

五个核心决策（AI 异步队列、全量存储、飞书作为平台载体、飞书 OAuth 登录、轮询替代 IDLE）都有明确依据，讨论过程中用户认可度高，没有出现方向性返工。决策质量整体良好。

### 方案完整性

**有一处遗漏**：初稿数据库表漏掉草稿箱（drafts），发件箱前端模块也未完整列出，被用户指出后补充。根因是设计数据库时没有系统比对 PRD 数据实体章节。

### PRD 理解准确性

飞书集成从"外部工具同步"升级为"平台载体"这个关键信息，是在技术讨论中逐步澄清的，PRD 本身没有明确写出。技术助手识别到了这个隐含信息并做了正确的架构调整（双重 OAuth 设计），理解准确。

### 流程改进

本次对话同步产出了三项流程改进：
1. 范本库（templates/）建立，解决新项目重复搭建骨架的问题
2. 三个角色新增主动复盘规范
3. planning-with-files-zh 与 SPRINT.md 的职责边界明确

### Feedback

- TRD 提交前需逐一核对 PRD 数据实体与数据库表对应关系（已同步至 roles/tech.md 自检项）

---

## 2026-04-05 · 程序员 · 工作流设计 & 模块 1-2 代码检查

### 交付内容

1. 建立 `templates/standards/backend.md` 和 `frontend.md`，沉淀历史踩坑，作为新项目生成 STANDARDS.md 的母本
2. 打通完整质量链路：product → tech → developer → 模块自检 → obstacle 记录 → 复盘 → feedback → templates 更新
3. 补全 `developer/mail-ai/docs/config/STANDARDS.md`（原本只有两条项目特有规则，现已包含完整后端和前端规范）
4. 修复模块 1-2 代码审查发现的致命/严重问题：
   - 新增 `SuccessInterceptor` 和 `GlobalExceptionFilter`，在 `main.ts` 全局注册
   - 新增 `UserResponseDto`，`loginWithCode` 和 `getUserById` 不再直接返回 User 实体，屏蔽 feishuOpenId / feishuUnionId
5. 中等及以下问题记入 `docs/backlog.md`（console.error、缺少唯一约束、缺少 @CreateDateColumn 等）

### 流程改进

本期同步产出的流程改进（均已写入对应角色文件）：

- **STANDARDS.md 生成时机**：开发启动前第一步生成，生成后必须重启会话，确保 CLAUDE.md 加载到完整规范再开始业务代码
- **模块类型判断**：三种类型（全栈/纯后端/纯前端）对应不同阶段，以表格形式固化进 developer.md
- **障碍记录→复盘→feedback 提升路径**：从 plan.md 障碍表 → 复盘回顾 → 推送 feedback → 协调者路由分发
- **三次失败协议**：防止对同一方案无限重试
- **会话恢复检查**：任务启动前检查 plans/ 目录 + git diff --stat，对照后再开始

### 根因说明

模块 1-2 代码审查发现一批问题，原因是这两个模块开发时 STANDARDS.md 为空，模块自检无有效基准。问题本身不是检查失误，是流程缺口——STANDARDS.md 生成时机的问题，已在本期流程改进中修掉。

### Feedback

无（本期无新增 feedback 条目）

---

## 2026-04-05 · 程序员 · Module 3 邮箱账户接入

### 交付内容

实现邮箱账户的增删查和 IMAP 连接测试，密码 AES-256 加密存库：

- `MailAccountsService`：create / findAll / remove / testConnection，含加密/解密、imapflow 连接测试、Logger
- `MailAccountsController`：4 个接口，类级 JWT 鉴权
- `CreateMailAccountDto`：完整 class-validator，含 `@IsDefined()`
- `MailAccountResponseDto`：不暴露 `passwordEncrypted`
- Entity：`email_address` 唯一约束，去掉错误的 `nullable: true`
- `MailAccountsModule`：import `AuthModule` 复用 `JwtModule`，不重复注册
- 顺手修复 Module 2 遗留的 TS 类型错误（`auth.service.spec.ts` 断言不存在的字段）

测试：39/39 通过，tsc 零错误。

### TRD 执行情况

完全按 TRD 4.2 节实现，4 个接口路径、入参、出参、鉴权均与文档一致，无偏离。

### 规范执行

**执行中发现的问题（均已修复）：**

1. **模块自检未主动触发**：模块完成后没有主动输出自检报告，直到用户追问才补做。根因是 executing-plans 流程里没有强制插入自检步骤
2. **code-reviewer subagent 未显式传入 STANDARDS.md 全文**：依赖 subagent 自行读取文件，违反 `roles/developer.md` 的 subagent 模式要求
3. **DTO 缺 `@IsDefined()`**：`imapPort`、`smtpPort`、`imapTls`、`smtpTls` 只有类型装饰器，缺少存在性声明
4. **Entity nullable 与 DTO 必填矛盾**：4 个必填列标了 `nullable: true`
5. **`MailAccountsModule` 冗余注册 JwtModule**：已有 AuthModule export，重复注册产生漂移风险
6. **testConnection catch 无服务端日志**：违反「不能只 catch 不记录」规范

### 流程改进

- **阶段复盘触发时机**：由「每期迭代全部模块完成后」改为「用户说这一阶段结束时」，已更新 `roles/developer.md`
- **模块自检流程**：待写入 `roles/developer.md`（见 Feedback）

### Feedback

- 2026-04-05 | 模块完成后未主动输出自检报告，subagent 模式下未将 STANDARDS.md 全文显式传入 reviewer prompt | 建议在 `roles/developer.md` 的 executing-plans 流程末尾强制插入自检步骤，subagent 模式需将 STANDARDS.md 全文写入 prompt

---

## 2026-04-05 · 程序员 · Module 4 IMAP 同步

### 交付内容

实现首次历史同步（30天）、5 分钟轮询增量同步、邮件入库、附件存盘：

- `ImapSyncService`：saveEmail / saveAttachments / startInitialSync / runIncrementalSync / syncAccount
- `ImapSyncScheduler`：`@Cron(EVERY_5_MINUTES)` 触发 runIncrementalSync
- `ImapSyncModule`：注册 TypeORM entities + Bull 队列，导出 ImapSyncService
- `MailAccountsService.create`：save 后 fire-and-forget 触发 startInitialSync
- `app.module.ts`：注册 BullModule.forRootAsync / ScheduleModule.forRoot / ImapSyncModule
- 共 27 个测试全部通过，tsc 零错误

### 规范执行

**执行中发现并修复的问题：**

1. **`simpleParser()` 返回类型**：mailparser 的 `simpleParser` 声明返回 `void & Promise<ParsedMail>`（类型库 bug），需显式 `as Promise<ParsedMail>` 并加 `if (!msg.source) continue` 防护
2. **`client.search()` 返回 `false | number[]`**：不能直接 `for...of`，需先 `if (!Array.isArray(uids) || !uids.length) return`
3. **`emailRepo.create({...})` TypeORM 重载歧义**：多字段对象触发歧义，需显式 `as DeepPartial<Email>`
4. **附件文件名路径穿越**：单用正则 `[^a-zA-Z0-9._-]` 不过滤 `..`，需先 `path.basename()` 再 regex
5. **`jest.mock('fs')` 破坏 TypeORM 初始化**：TypeORM 通过 `glob → path-scurry` 依赖 `fs.statSync`，完全替换 fs 导致初始化崩溃；解决：`jest.requireActual('fs')` 保留真实 fs，只 override `promises.mkdir/writeFile`

### 流程改进

无新增流程改进（Module 3 已建立的自检流程本期正常执行）

### 技术沉淀（已同步至 templates/standards/backend.md）

- Jest mock 类型转换：`(X as unknown as jest.Mock)` 双重转换模式
- `jest.mock('fs')` 安全写法：`jest.requireActual` 保留真实模块
- 文件路径安全：`path.basename()` + regex 两步组合，单独 regex 不足以防路径穿越

### Feedback

无

---

## 2026-04-07 · 程序员 · 整体联调第二轮

### 背景

一至四期全部完成后进入整体联调。本轮按模块逐一测试，发现功能 Bug → 修复 → 完成深度代码审查 → 修复所有安全和质量问题。

### 功能 Bug 修复

| 模块 | 问题 | 修复 |
|------|------|------|
| 收件箱 | 智能/标签筛选不应在收件箱出现 | 移除，仅保留日期筛选 |
| 收件箱 | 日期筛选无效 | 补全 `dateFrom`/`dateTo` DTO 字段 + WHERE 条件 |
| 收件箱 | 两账户邮件混在一起 | X-Account-Id 传递 + 切换账户时重新 fetch |
| 收件箱 | 已读后角标不实时更新 | `handleRead` 调用 `decrementUnread()` |
| 账户切换 | 刷新后回到第一个账户 | localStorage 持久化 `currentAccountId` |
| 账户切换 | 切换器位置在顶部 | 移到左侧导航栏底部，用 `el-popover` 下拉 |
| 同步 | 无手动接收入口 | 收件箱工具栏加"接收邮件"按钮 |
| 同步 | 不知道同步多少封/已同步多少 | 后端 `syncProgress` Map 实时记录，前端 3s 轮询 |
| 同步 | 账户 2 卡在同步中 | 补 TLS 配置、连接超时、error 事件监听、并发锁 |
| 写信 | 发件人不随账户切换 | `accountId` 全链路传递（send / reply / forward） |
| 写信 | 全部回复触发的是单独回复 | 增加 `isReplyAll` ref 区分并传递给 ComposeModal |
| 写信 | 点附件按钮跳出页面 | 所有 `<button>` 补 `type="button"`；实现完整附件上传流程 |

**决策**：手动停止同步功能经确认不需要，不做。

### 代码审查修复（安全 + 质量）

| 级别 | 问题 | 修复 |
|------|------|------|
| Critical | 路径穿越：任意服务器文件可作附件外发 | 发送前用 `uploadDir` 白名单校验路径 |
| Critical | 上传接口返回绝对路径泄露目录结构 | 改返回 `path.relative(cwd, file.path)` |
| High | 上传接口无文件类型限制 | `fileFilter` 白名单（pdf/doc/image/zip 等） |
| High | 临时附件发送后不清理 | `send/reply/forward` 统一在 `finally` 里 `fs.unlink` |
| High | `syncAccountWithLock.finally` 未清 `syncProgress` | 补 `this.syncProgress.delete(account.id)` |
| High | `loadUnreadCount` 拉 200 封邮件只为数数 | 新增 `GET /emails/unread-count` 专用接口 |
| Medium | 回复/转发附件静默丢失 | DTO + Service + 前端全链路支持附件 |
| Medium | `dateTo` 过滤排除当天邮件 | 补 `T23:59:59`（`findAll` 和 `search` 两处） |
| Medium | 同步轮询无上限，后端卡住时无限请求 | 计数器上限 100 次（≈5 分钟）自动停止 |
| Low | `toggleToolbar` 空桩按钮 | 删除按钮和函数 |
| Low | `findAll` 的 `emailCount` 恒为 0 | 改 `Promise.all` 并发查各账户邮件数 |

### 技术沉淀

- **附件路径安全两层缺一不可**：上传接口只返回相对路径；发送前 `path.resolve + startsWith(uploadDir)` 双重校验
- **临时文件生命周期**：compose-tmp 由 `send-mail.service` 在 `finally` 里统一清理
- **count 类查询用专用接口**：`getCount()` 而非列表接口 filter，超出 pageSize 时结果不准

### Feedback

- 附件功能涉及上传、存储、发送三层各自的安全责任，下次实现类似功能需在设计阶段列安全 checklist，不等代码审查
- `// 预留` 空桩代码违反编码规范，禁止提交，已写入本轮修复中删除
