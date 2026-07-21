# KX — 页面描述语言

**K**nowledge e**X**change · 让 AI 读懂业务，让代码生于架构

[![Version](https://img.shields.io/badge/version-1.2-blue?style=flat-square)]()
[![Language](https://img.shields.io/badge/DSL-Declarative-orange?style=flat-square)]()
[![AI Native](https://img.shields.io/badge/AI-Native-brightgreen?style=flat-square)]()
[![Target](https://img.shields.io/badge/target-Vue_3-42b883?style=flat-square)](https://vuejs.org/)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)]()

---

## 为什么选择 KX？

- **业务优先**：用产品语义描述页面，而非实现细节。
- **AI 友好**：结构化的 `@指令` 语法，AI 可直接解析并转换为代码。
- **显式数据**：`@state`、`@prop`、`@api -> state` 让数据流清晰可见。
- **交互优先**：`@hover` 双模式、`@popover`、`@anchor`、`@delay` 均为一等公民。
- **权限原生**：`@login` 和 `@permission` 作为指令，AI 自动生成守卫逻辑。
- **单一事实源**：一个 `.kx` 文件 = 布局 + 数据 + 逻辑 + 权限 + 业务规则。

---

## 快速上手

### 第一步：阅读规范

完整语法规范见 [SPEC.md](./SPEC.md)。

### 第二步：创建项目目录结构

```
项目结构：
├── guide/                      ← 需求与设计文档根目录
│   ├── index.kx                ← 入口文件：@ref 引用所有文件
│   ├── layouts/
│   │   └── main.kx             ← 全局布局
│   ├── pages/
│   │   ├── home.kx             ← 首页
│   │   ├── mine.kx             ← 个人中心
│   │   └── detail.kx           ← 详情页
│   ├── models/
│   │   ├── user.kx             ← 数据模型：User
│   │   ├── work.kx             ← 数据模型：Work
│   │   └── api.kx              ← API 接口定义
│   └── components/
│       └── shared.kx           ← 共享组件（浮窗等）
├── src/                        ← 代码目录（AI 生成）
└── guide/style.md              ← 前端风格说明（配色、字体、间距）
```

### 第三步：用 @ref 串联文件

在 `index.kx` 中通过 `@ref` 引用所有文件，AI 自动定位：

```kx
// index.kx — 项目入口
@ref ./layouts/main.kx
@ref ./pages/home.kx
@ref ./pages/mine.kx
@ref ./pages/detail.kx
@ref ./models/user.kx
@ref ./models/work.kx
@ref ./models/api.kx
@ref ./components/shared.kx
```

### 第四步：编写 `.kx` 文件

按目录分类编写各模块文件，`models/` 下定义数据模型：

```kx
// models/user.kx
@model User {
  @field id: number
  @field username: string
  @field avatar: string
  @field role: string
}

// models/api.kx
@model API {
  @api GET /api/v1/works -> works
  @api POST /api/v1/works/:id/like -> void
}
```

### 第五步：投喂给 AI

---

## 核心概念

| 指令 | 作用 | 生成结果 |
|:---|:---|:---|
| `@page` | 页面路由 + 元信息 | 路由定义 + 视图组件 |
| `@layout` / `@slot` | 布局骨架 + 插槽 | 布局包裹组件 |
| `@api` | 接口请求 + 数据绑定 | API 调用 + 响应式状态 |
| `@mutation` | 本地状态变更 | 乐观更新代码 |
| `@render` | 条件渲染 | `v-if` / `v-else` |
| `@navigate` | 导航跳转 | `router.push()` |
| `@login` / `@permission` | 权限控制 | 路由守卫 / `v-if` 校验 |
| `@hover` / `@popover` | 悬浮交互 | 悬浮状态 + 浮窗组件 |
| `@note` | 业务约束（AI 强制识别） | 代码注释 + 辅助逻辑 |

---

### 语法全景

**结构指令：**

| 语法 | 作用 |
|:---|:---|
| `@page` | 页面路由定义 |
| `@slot` | 页面区域划分 |
| `@layout` | 全局布局骨架 |

**组件指令（v1.2 块级语法）：**

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@button` | 按钮 | `@button 提交 { @note 提交表单 }` |
| `@list` | 瀑布流/列表 | `@list 作品流 { @api GET /works -> list }` |
| `@card` | 卡片 | `@card 作品卡片 (v-for: item in list)` |
| `@tab` | 选项卡 | `@tab 频道 { @state active: string = 'hot' }` |
| `@header` | 顶部栏 | `@header 主导航` |
| `@sidebar` | 侧边栏 | `@sidebar 左侧菜单` |
| `@modal` | 弹窗 | `@modal 确认删除 { @note 二次确认 }` |
| `@popover` | 悬浮浮窗 | `@popover 用户信息卡` |
| `@badge` | 角标 | `@badge 新品 { @render when: item.is_new }` |
| `@avatar` | 头像 | `@avatar 用户头像 (bind: user.avatar)` |
| `@stat` | 统计数字 | `@stat 粉丝数 (bind: followCount)` |
| `@media` | 图片/视频 | `@media 主图 (bind: work.image_url)` |
| `@form` | 表单 | `@form 登录表单` |
| `@input` | 输入框 | `@input 搜索框` |
| `@menu` | 菜单项 | `@menu 个人中心` |
| `@action` | 操作按钮区 | `@action 工具栏` |
| `@empty` | 空状态 | `@empty 无数据` |
| `@skeleton` | 骨架屏 | `@skeleton 列表占位 (count: 6)` |
| `@toast` | 轻提示 | `@toast 操作成功` |
| `@text` | 文本展示 | `@text 用户名 (bind: user.name)` |
| `@logo` | Logo | `@logo 网站Logo` |
| `@banner` | 横幅轮播 | `@banner 首页轮播` |
| `@detail` | 详情容器 | `@detail 作品详情` |
| `@author` | 作者信息区 | `@author 作者卡片` |

**数据与逻辑指令：**

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@api` | 接口请求 + 绑定 | `@api GET /works -> list` |
| `@mutation` | 本地状态变更 | `@mutation set list[idx].liked = true` |
| `@sync` | 计算属性同步 | `@sync = myWorks.length` |
| `@prop` | 组件属性 | `@prop item: Work` |
| `@state` | 本地状态 | `@state loading: boolean = false` |
| `@navigate` | 路由跳转 | `@navigate click -> /detail` |
| `@permission` | 权限校验 | `@permission work:create` |
| `@login` | 登录守卫 | `@login` |
| `@render` | 条件渲染 | `@render when: loading` |
| `@note` | 业务说明（AI 必读） | `@note 仅 VIP 可见` |

**悬浮交互（v1.2 双模式）：**

| 模式 | 写法 | 适用场景 |
|:---|:---|:---|
| **自动托管** | `@hover -> @popover 浮窗名` | 静态内容、无接口、零延迟需求 |
| **手动控制** | `@hover { @state s=true @delay show: 300ms }` + `@leave { ... }` | 异步加载、防误触、精确定位 |

---

## 完整示例

完整 4 场景示例见 [example.kx](./example.kx)（全局布局 / 精选首页 / 作品详情 / 我的空间）。

---

## 写作规范

1. 一个 `.kx` 文件对应一个页面或独立模块。
2. 用 `@ref` 在 `index.kx` 中引用所有文件，AI 据此自动定位。
3. 用 `@note` 显式记录业务规则 — AI 将其视为强制约束。
4. 用 `@api -> state`、`@mutation`、`@sync` 保持数据流显式。
5. 每个 `@list` 内部使用 `@loading` / `@empty` / `@error` 状态容器。
6. 需要登录的操作或内容添加 `@login`。
7. 交互逻辑（悬浮、弹窗、导航）用 `@hover` / `@popover` / `@navigate`。

---

## 路线图

- [x] v1.0 — 基础语法（`@component` 前缀）
- [x] v1.1 — 布局继承 + 乐观更新 + 路由参数 + 状态容器
- [x] v1.2 — 块级语义 `@<类型>` + `@note` + `@hover` 双模式 + `@login` 三模式
- [ ] v1.3 — `@typedef` 类型定义系统
- [ ] v1.4 — React / Svelte 映射模板
- [ ] v2.0 — VS Code 语法高亮插件（见 [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension)）
- [ ] v2.1 — KX → 代码的完整编译器

---

## 关联仓库

- [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension) — `.kx` 文件的 VS Code 语法高亮扩展

---

## 参与贡献

欢迎参与：

- 完善 KX 规范与示例
- 增强 VS Code 语法支持
- 优化文档与写作指南

请提交 Issue 或 Pull Request。

---

## 许可证

MIT 许可证
