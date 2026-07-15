<div align="center">

# KX — Page Definition Language

**K**nowledge e**X**change · 让 AI 读懂业务，让代码生于架构

[![Version](https://img.shields.io/badge/version-1.2-blue?style=flat-square)](https://github.com/yijiu2025/kx-lang/releases)
[![Language](https://img.shields.io/badge/DSL-declarative-orange?style=flat-square)](SPEC.md)
[![AI Native](https://img.shields.io/badge/AI-native-brightgreen?style=flat-square)](README.en.md)
[![Target](https://img.shields.io/badge/target-Vue_3-42b883?style=flat-square)](https://vuejs.org/)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

[English](README.en.md) · [中文](README.zh.md)

</div>

---

## 🌐 Language / 语言

- [English](README.en.md)
- [中文](README.zh.md)

---

KX is a lightweight **declarative language** for describing page architecture so that AI can reliably generate Vue 3 code from it. A single `.kx` file captures structure, data flow, interaction, permissions, and business constraints — replacing scattered prompts with a single source of truth.

KX 是一种轻量级**声明式语言**，用 `.kx` 文件描述页面架构，让 AI 能可靠地生成 Vue 3 代码。结构、数据流、交互、权限和业务约束汇聚于一处，取代零散的提示词。

---

## ⚡ Quick Start / 快速上手

```kx
// .kx — page specification
@page /discover (发现页) extends AppLayout {
  @icon SearchOutlined
  @meta title="发现" description="推荐热门作品与搜索结果"

  @slot main {
    @header 搜索栏 {
      @input 搜索框 (bind: searchQuery)
      @action 筛选区 {
        @button 推荐 { @state tab: string = 'recommend' }
        @button 最新 { @state tab = 'latest' }
      }
    }

    @list 作品流 {
      @api GET /api/v1/works (query: page, keyword: searchQuery, tab: tab) -> works
      @render when: works.length > 0

      @card 作品卡片 (v-for: item in works) {
        @prop item: Work
        @media 封面图 (bind: item.cover_url)
        @text 标题 (bind: item.title)
        @avatar 作者 (bind: item.author.avatar)

        @button 喜欢 {
          @api POST /api/v1/works/:id/like (body: { workId: item.id })
          @mutation set works.find(w => w.id === item.id).liked = true
          @login
        }

        @hover -> @popover 作品预览
      }

      @empty 暂无作品 {
        @text 没有找到相关作品
        @button 返回首页 { @navigate click -> / }
      }

      @loading { @skeleton 卡片骨架 (count: 6) }
    }
  }
}
```

AI generates a complete Vue 3 application from the spec above:

| KX 输入 | AI 输出 |
|:---|:---|
| `@page /discover (发现页)` | `router/index.ts` 路由 + `HomeView.vue` |
| `@card 作品卡片 { @prop item: Work … }` | `components/PoseCard.vue` 子组件 |
| `@api GET /api/v1/works … -> works` | `api/works.ts` 接口封装 |
| `@button 喜欢 { @mutation … @login }` | 点赞逻辑 + 登录守卫 |

---

## ✨ Core Features / 核心特性

| | Feature | 说明 |
|:---:|:---|:---|
| 📐 | **Declarative Structure** | `@page` `@layout` `@slot` 描述页面骨架，层级即语义 |
| 🔗 | **Data & API Binding** | `@api` `@state` `@param` 声明数据来源与流向 |
| 🔄 | **State & Mutations** | `@mutation` `@sync` 显式表达状态变更与计算属性 |
| 🎨 | **23 Component Types** | 从 `@header` 到 `@form`，覆盖布局、内容、交互、反馈 |
| 🧭 | **Navigation & Events** | `@navigate` `@event` 覆盖路由跳转与 DOM 事件 |
| 💬 | **Interaction & Feedback** | `@hover` `@popover` `@modal` `@toast` 语义化交互 |
| 🔒 | **Auth & Permissions** | `@login` `@permission` 声明式权限守卫 |
| 🤖 | **AI-Native Design** | `@note` 让业务约束对 AI 可见、可执行 |

---

## 📖 Documentation / 文档导航

| Resource | 说明 |
|:---|:---|
| [SPEC.md](SPEC.md) | KX v1.2 语言规范（完整语法 + 生成映射表） |
| [example.kx](example.kx) | 4 个实战示例（布局 / 首页 / 详情 / 我的） |
| [README.zh.md](README.zh.md) | 中文完整文档 |
| [README.en.md](README.en.md) | English complete docs |
| [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension) | VS Code 语法高亮扩展 |

---

## 🔗 Related Repositories / 关联仓库

- [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension) — VS Code syntax highlighting for `.kx` files
- [kx-lang](https://github.com/yijiu2025/kx-lang) — specification, examples, and docs (this repository)

---

## 🆚 Why KX? / 为什么选择 KX？

| Aspect | Traditional Development | KX |
|:---|:---|:---|
| Page definition | Hand-written Vue SFCs | Declaring intent in `.kx` |
| AI collaboration | Ad-hoc prompts | Structured, parseable spec |
| Business logic | Scattered in组件 code | Centralized via `@note` |
| State management | Imperative mutations | Declarative `@mutation` |
| Component choice | Manual coding | 23 semantic directives |
| Maintenance | Code review per change | Update spec, regenerate |

---

## 📋 Roadmap / 路线图

- [x] **v1.0** — 基础指令集：`@page` `@slot` `@button` `@api` `@navigate`
- [x] **v1.1** — 新增 `@mutation` `@sync` `@render` `@state` `@prop` `@param`
- [x] **v1.2** — 新增 `@hover` `@leave` `@anchor` `@position` `@delay` `@permission` `@login` `@event` `@form`；扩展组件至 23 种
- [ ] **v2.0** — 服务端函数 `@server`、WebSocket `@ws`、图形绑定 `@graph`、自定义指令扩展
- [ ] **v2.1** — 多端适配 `@platform`、AI 辅助生成器 `@ai`、设计稿互转 `@pen`

---

## 🤝 Contributing / 参与贡献

欢迎提交 Issue 或 PR：

Welcome contributions of all kinds:

- 完善 `.kx` 写法和示例 / Improve `.kx` examples and patterns
- 增强语法支持 / Enhance syntax coverage in `kx-lang-extension`
- 优化文档与规范 / Refine documentation and the KX specification

---

## 📄 License / 许可证

[MIT](LICENSE) · Copyright (c) 2025 KX Language Contributors
