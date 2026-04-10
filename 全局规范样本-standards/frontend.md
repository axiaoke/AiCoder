# 前端编码规范（通用模板）

> 项目启动时复制此文件到 `docs/config/STANDARDS.md`，按项目实际情况增删。
> 通用规则在此维护；项目特有规则（色彩变量、断点值、业务状态枚举）写在项目 STANDARDS.md 末尾，不要改这里。

---

## 设计系统（强制）

- 禁止硬编码颜色值（如 `color: #333`），统一使用 SCSS 变量
- 禁止硬编码间距（如 `margin: 20px`），统一使用间距变量（8px 网格）
- 项目启动时在 `src/styles/variables.scss` 中定义色彩和间距变量，后续所有样式引用变量

### 间距（8px 网格，不得自造数值）

```scss
// 项目初始化时在 variables.scss 中定义
$space-xs: 4px;  $space-sm: 8px;  $space-md: 16px;
$space-lg: 24px; $space-xl: 32px; $space-xxl: 48px;
```

### 字号

```scss
$font-size-h1: 32px; $font-size-h2: 24px; $font-size-h3: 18px;
$font-size-body: 16px; $font-size-caption: 14px; $font-size-small: 12px;
```

---

## Vue 组件规范

### 文件结构顺序（强制）

```
<template> → <script setup lang="ts"> → <style scoped lang="scss">
```

### 文件命名（强制）

- 统一 PascalCase，如 `TaskDetail.vue`
- 禁止 kebab-case 命名（如 `task-detail.vue`）

### 类型共享规则

- 只在单个组件内用 → 可以定义在组件内
- 跨两个及以上组件使用 → 提取到对应的 `@/api/*.ts` 文件顶部并 export
- 全局通用的枚举 / 常量 → 放入 `@/types/enums.ts`
- 禁止在组件内重复定义已在 store / api / enums 中导出的类型

---

## SCSS 规范

### 引入方式（强制）

```scss
@use '@/styles/variables.scss' as *;   // 正确
@import '@/styles/variables.scss';      // 错误，已废弃
```

- 禁止混用 `@use` 和 `@import`
- 所有 `<style lang="scss">` 块（含 scoped）必须在块内独立声明 `@use`，不继承父级

---

## 异步请求规范（强制）

```ts
try {
  const res = await getSomething(id)
  data.value = res.data
} catch {
  ElMessage.error('加载失败，请重试')
} finally {
  loading.value = false
}
```

- 禁止只写 `finally` 不写 `catch`
- loading 状态统一在 `finally` 中重置

---

## 表单验证规范（强制）

```ts
// Promise 式（正确）
try {
  await formRef.value.validate()
  submitting.value = true
  // 提交逻辑
} catch {
  // validate 失败自动走这里
} finally {
  submitting.value = false
}

// 禁止回调式
// formRef.value.validate(async (valid) => { ... })
```

---

## Element Plus 使用规范

- 详情抽屉宽度：`500–520px`，移动端 `width: 100%`
- 编辑表单弹窗宽度：`420–500px`，移动端 `width: 95vw`
- 所有 `el-dialog` 必须加 `align-center`（移动端默认贴顶，体验差）
- 简单确认用 `ElMessageBox.confirm`，type `'warning'`，不需要单独写弹窗组件
- 删除操作：必须经过 `ElMessageBox.confirm`
- 表单 `label-width`：主表单 `90px`，简单表单 `80px`
- 日期选择器、下拉框：统一加 `style="width: 100%"`
- 禁止引入新的第三方 UI 库（统一用 Element Plus）

### Element Plus 已知陷阱（禁止重犯）

- **禁止**在 El Plus 组件 tag 上用 inline `style` 或 CSS class 直接控制宽度（用 `div wrapper + :deep(.el-select) { width: 100% !important }` 代替；multiple 模式下尤其必要）
- **禁止**用 CSS class / 媒体查询控制 El Plus 组件显隐（用 `v-if` / `v-show` 代替）
- **禁止**用 `el-dropdown` 作为纯图标触发器（会附加视觉样式；改用 `el-popover` 的 `#reference` slot）
- **禁止**将 `el-dialog` 的 `title` prop 设为空字符串（会破坏宽度计算；保留非空值，用 `.el-dialog__title { display: none }` 隐藏文字）
- **禁止** el-select option 的 `value` 使用空字符串（El Plus 视 `''` 为"未选择"，会显示 placeholder；改用 `'all'` 等有意义字符串）

---

## 响应式规范

- 响应式断点由项目在 `variables.scss` 中定义（如 `$breakpoint-mobile: 768px`）
- 所有弹窗：移动端 `width: 95vw`
- 所有右侧抽屉：移动端 `width: 100%`
- 输入框 `font-size`：移动端不低于 `16px`（低于 16px 在 iOS 触发页面放大）
- 输入框宽度：移动端统一 `width: 100%`，不得写死像素值
- 按钮最小点击区域：`44px × 44px`，禁止用 `margin: -Npx` 扩大点击区域（用 `min-width: 44px; min-height: 44px` + flex 居中代替）
- 禁止在包含 `overflow-x: auto` 的 flex 祖先上设置 `overflow: hidden`（会压缩滚动区域至零宽）
- 禁止对内容容器写死像素宽度（用 `max-width` + `width: 100%` 代替，移动端自适应）
- 表格必须用 `overflow-x: auto` 的 wrapper 包裹，防止移动端横向溢出
- 长文本截断统一用 `overflow: hidden; text-overflow: ellipsis; white-space: nowrap`
- Flex / Grid 多列布局必须有移动端单列降级（`flex-direction: column` 或 `grid-template-columns: 1fr`）
- 所有 `<img>` 必须设 `max-width: 100%`，防止溢出容器

---

## 布局规范

- 禁止将展开型输入框（搜索、人员筛选等）放在 flex 同行容器内（展开后撑破布局；改用 `v-if` 控制的独立展开行）

---

## Vite ESM 模块规范（强制）

- **禁止在 Vite ESM 项目中使用 `require()`**：Vite ESM 环境下 `require` 未定义，运行时抛出 `ReferenceError`，且可能被 `try/catch` 静默吞掉，导致逻辑错误难以发现
- 循环依赖解决方案：通过 `localStorage` 桥接或重构依赖关系，禁止用 `require()` 绕过

## Pinia Store 规范（强制）

- **跨页面刷新的状态必须持久化**：需要在刷新后保留的 store 字段，初始化时从 `localStorage` 读取作为初始值，写入时同步更新 `localStorage`
- **退出登录必须重置所有业务 store**：`logout()` 除清除 auth store 外，必须调用所有业务 store 的 `reset()`，防止脏数据在下次登录时残留

## catch 块规范（强制）

- **catch 块必须透传后端错误信息**：`ElMessage.error((err as Error).message || '操作失败')` — 禁止写死错误文案，后端返回的具体原因必须展示给用户

## 代码整洁

- 禁止提交含 `// TODO`、`// FIXME` 的未完成代码
- 禁止未使用的 import
- 禁止遗留 `console.log`
- 禁止提交注释掉的废弃代码块
