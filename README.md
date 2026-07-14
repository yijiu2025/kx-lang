<div align="center">

# KX — 页面描述语言

**K**nowledge e**X**change · 让 AI 读懂业务，让代码生于架构

[![Version](https://img.shields.io/badge/version-1.2-blue?style=flat-square)]()
[![Language](https://img.shields.io/badge/DSL-Declarative-orange?style=flat-square)]()
[![AI Native](https://img.shields.io/badge/AI-Native-brightgreen?style=flat-square)]()
[![Target](https://img.shields.io/badge/target-Vue_3-42b883?style=flat-square)](https://vuejs.org/)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)]()

[English](#english) · [文档](SPEC.md) · [示例](app.kx) · [参与贡献](#参与贡献)

</div>

---

## KX 是什么？

**KX** 是一种专为 **AI 代码生成** 设计的声明式页面描述语言。

用一段 `.kx` 文件描述页面的结构、数据流和交互逻辑，AI 就能生成类型安全、生产就绪的 Vue 3 代码。

---

## ⚡ 30 秒看懂

```kx
@page /home (首页) {
  @slot main {
    @list Waterfall {
      @api GET /api/works (query: page) -> works

      @card PoseCard (v-for: item in works) {
        @prop item: Work
        @avatar 作者 (bind: item.author)

        @action 点赞 {
          @api POST /works/:id/like
          @mutation works[item].liked
          @login
        }
      }

      @empty EmptyState { @render unless: works.length }
    }
  }
}
```

上面这段 `.kx` 代码，AI 会自动生成：

- 📄 `views/HomeView.vue` — 页面模板 + 样式
- 📄 `components/cards/PoseCard.vue` — 卡片子组件
- 📄 `api/works.ts` — 接口请求函数
- 📄 `router/index.ts` — 路由注册

> 数据来源、用户操作、条件渲染、权限校验 —— 全部在一处声明完毕。

---

## 🛡️ `@login` 登录守卫

```kx
@avatar 头像 {
    @click -> /mine
    @login      // 未登录 → 跳登录页 → 登录成功后返回 /mine
}
```

三种模式覆盖所有场景：

| 模式 | 写法 | 效果 |
|:---|:---|:---|
| 隐藏组件 | `@permission code` | 无权限则 `v-if="false"`，完全不可见 |
| **导航守卫** | `@login` + `@click` | 拦截 → 跳登录 → 登录后继续跳转 |
| 内容保护 | `@login` 单独使用 | 未登录显示登录引导卡，已登录显示真实内容 |

---

## 🖱️ 悬浮交互（双模式）

```kx
/* 模式一：一行搞定 — 静态浮窗 */
@avatar 头像 { @hover -> @popover 用户卡 @click -> /mine }

/* 模式二：按需加载 + 延迟防误触 */
@card 作品卡片 (v-for: item in works) {
  @state show = false
  @hover { @state show = true @delay show: 300ms }
  @leave { @delay hide: 200ms }
  @popover 详情预览 {
    @render when: show
    @anchor @card 作品卡片
    @api GET /works/:id/detail (bind: item.id) -> detail
    @media 大图 (bind: detail.hd_image)
    @button 快速点赞 { @api POST /works/:id/like @login }
  }
}
```

| 模式 | 写法 | 适用场景 |
|:---|:---|:---|
| **自动托管** | `@hover -> @popover 浮窗名` | 静态内容、无接口、零延迟需求 |
| **手动控制** | `@hover { @state s=true @delay show: 300ms }` + `@leave { ... }` | 异步加载、防误触、精确定位 |

---

## 为什么需要 KX？

| | 传统方式 | KX + AI |
|:---|:---|:---|
| **描述一个页面** | 画原型 → 写 `.vue` → 写 `.ts` → 调接口 → 改 Bug | 写一个 `.kx` 文件 → 投喂给 AI → 运行 |
| **修改需求** | 改 `.vue`、`.ts`、`.scss` 三个文件 | 改一行 `.kx` 描述，AI 自动同步 |
| **新人理解代码** | 翻遍组件、Store、API 层才能理清数据流 | 一个 `@api` + `@mutation` 看清全链路 |
| **产出效率** | 1 个页面 / 天 | 1 个页面 / 10 分钟 |

---

## ✨ 特性

- **🧠 双向可读** — 产品经理能看懂，AI 能精准执行
- **🧩 布局继承** — `@layout` 定义一次全局骨架，子页面只写差异
- **📐 块级语义（v1.2）** — `@<类型> <名称> { ... }` 统一结构，零歧义
- **📝 结构化说明** — `@note` 让业务逻辑脱离"代码注释"，AI 强制识别
- **⚡ 乐观更新** — `@mutation` 显式声明本地状态变更，AI 自动生成回滚逻辑
- **🎯 类型安全** — `@prop`、`@state`、`@api -> state` 显式绑定，禁止 `any`
- **🛡️ 权限内建** — `@permission` + `@login` 原生支持，AI 自动生成守卫逻辑
- **🖱️ 悬浮交互（v1.2）** — `@hover` 双模式：一行声明搞定静态浮窗，状态+延迟+锚点搞定高级交互
- **🏷️ 元数据** — `@icon` + `@meta`，AI 生成路由时自动注册标题和图标

---

## 📚 语法全景

### 结构指令

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@page` | 定义页面路由 | `@page /home (首页)` |
| `@slot` | 页面区域划分 | `@slot main (role: main)` |
| `@layout` | 全局布局骨架 | `@layout MainLayout { ... }` |

### 组件指令（v1.2 块级语法）

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@button` | 按钮 | `@button 提交 { @note 提交表单 }` |
| `@list` | 瀑布流/列表 | `@list 作品流 { @api GET /works -> list }` |
| `@card` | 卡片 | `@card 作品卡片 (v-for: item in list)` |
| `@tab` | 选项卡 | `@tab 频道Tab { @state active: string = 'hot' }` |
| `@header` | 顶部栏 | `@header 主导航` |
| `@sidebar` | 侧边栏 | `@sidebar 左侧菜单` |
| `@modal` | 弹窗 | `@modal 确认删除 { @note 二次确认 }` |
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
| `@popover` | 悬浮浮窗 | `@popover 用户信息卡` |

### 数据与逻辑指令

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@api` | 接口请求 + 数据绑定 | `@api GET /works -> list` |
| `@mutation` | 本地状态变更 | `@mutation list[item].liked` |
| `@sync` | 计算属性同步 | `@sync = list.length` |
| `@prop` | 组件属性声明 | `@prop item: Work` |
| `@navigate` | 路由跳转 | `@navigate click -> /detail` |
| `@permission` | 细粒度权限校验 | `@permission work:create` |
| `@login` | 登录路由守卫 | `@login`（拦截跳转 / 内容保护） |
| `@render` | 条件渲染 | `@render when: loading` |
| `@note` | 业务说明（AI 必读） | `@note VIP 可见` |

### 悬浮交互指令（v1.2 双模式）

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@hover -> @popover` | 自动托管悬浮 | `@hover -> @popover 用户卡` |
| `@hover { ... }` | 手动控制 — 进入 | `@hover { @state show = true @delay show: 300ms }` |
| `@leave { ... }` | 手动控制 — 离开 | `@leave { @delay hide: 200ms }` |
| `@anchor` | 浮窗定位锚点 | `@anchor @avatar 头像` |
| `@position` | 浮窗显示方位 | `@position bottom-left` |

> 完整语法规范见 [SPEC.md](SPEC.md)

---

## 🔄 工作流程

```
┌─────────────┐      ┌──────────────┐      ┌─────────────────┐
│  编写 .kx    │ ──▶  │  AI 读取解析   │ ──▶  │  生成 Vue 3 代码  │
│  架构描述    │      │  SPEC.md 规范  │      │  组件/API/路由    │
└─────────────┘      └──────────────┘      └─────────────────┘
       ▲                                            │
       │                                            │
       └────────── 需求变更时修改 .kx ──────────────┘
```

### 配合 AI 使用的步骤

1. **投喂规范** — 把 `SPEC.md` 放入项目根目录，在 `.cursorrules` 中加入提示词
2. **编写架构** — 用 KX 语法描述页面，多用 `@note` 补充业务逻辑
3. **生成代码** — 在 Cursor 中 `@` 引用 `.kx` 文件，发送生成指令
4. **迭代维护** — 需求变更只改 `.kx`，AI 自动同步所有生成文件

---

## 🗂️ 项目结构

```
.
├── SPEC.md              # KX 语言完整规范（v1.2）
├── app.kx               # PoseCraft 全局架构（10 个页面完整示例）
└── README.md            # 本文件
```

### app.kx 包含的页面

| 页面 | 路由 | 说明 |
|:---|:---|:---|
| 主页 | `/` | 顶部导航 + 侧边栏 + 频道 Tab + 瀑布流 |
| 精选 | `/featured` | 继承主页布局，仅主内容区变化 |
| 推荐 | `/recommend` | 推荐算法数据流 |
| 附近 | `/nearby` | 含距离角标 |
| 关注 | `/following` | 关注用户数据 |
| 朋友 | `/friends` | 朋友作品流 |
| 我的 | `/mine` | 个人中心，含 Tab 数字同步 |
| 作品详情 | `/work/:id` | 详情页，含互动操作 |
| 编辑器 | `/editor` | 模板编辑器 |
| 拍照 | `/camera` | 上传 + 创建作品 |

---

## 📋 路线图

- [x] v1.0 — 基础语法 `@component` 前缀
- [x] v1.1 — 布局继承 + 乐观更新 + 路由参数 + 状态容器
- [x] v1.2 — 块级语义 `@<类型> { ... }` + `@note` + `@hover` 双模式 + `@login` 三模式
- [ ] v1.3 — `@typedef` 类型定义系统
- [ ] v1.4 — React / Svelte 映射模板
- [ ] v2.0 — VSCode 语法高亮插件
- [ ] v2.1 — KX → 代码的完整编译器

---

## 🤝 参与贡献

KX 仍处于早期孵化阶段，欢迎以下贡献：

- 🐛 语法设计建议（如何让 AI 理解更精准）
- 🔧 更多前端框架的代码映射模板（React / Svelte / Flutter）
- 🎨 VSCode / JetBrains 语法高亮插件
- 📖 文档翻译与完善

请通过 Issue 或 Pull Request 参与。

---

## English

**KX (Knowledge eXchange)** is a declarative page-description language designed for **AI code generation**.

Describe a page's structure, data flow, and interactions in a `.kx` file — AI tools like Cursor or Claude can then generate type-safe, production-ready Vue 3 code from it.

```kx
@page /home {
  @slot main {
    @list Waterfall {
      @api GET /api/works -> works
      @card PoseCard (v-for: item in works) {
        @prop item: Work
        @action 点赞 { @api POST /works/:id/like @login }
      }
    }
  }
}
```

→ Generates: `HomeView.vue`, `PoseCard.vue`, `api/works.ts`, `router/index.ts`

### Key Features

- **Block-level syntax** — `@<type> <Name> { ... }` unified structure
- **`@note` directive** — Business rules as first-class citizens, AI-enforced
- **`@hover` dual mode** — One-liner for simple popovers, full state control for advanced cases
- **`@login` guard** — Auto-generate login redirect / content protection logic
- **`@permission`** — Fine-grained access control, AI generates `v-if` automatically

See [SPEC.md](SPEC.md) for the full language specification.
