<div align="center">

# KX — Page Definition Language

**K**nowledge e**X**change · 让 AI 读懂业务，让代码生于架构

<!-- prettier-ignore -->
[![Version](https://img.shields.io/badge/version-1.2-5B6DEF?style=for-the-badge&logo=bookmark&logoColor=white)](https://github.com/yijiu2025/kx-lang/releases)
[![AI Native](https://img.shields.io/badge/AI-Native-00D26A?style=for-the-badge&logo=openai&logoColor=white)](SPEC.md#三ai-生成映射表)
[![Target](https://img.shields.io/badge/Target-Vue_3-42b883?style=for-the-badge&logo=vue.js&logoColor=white)](https://vuejs.org/)
[![License](https://img.shields.io/badge/License-MIT-E12B89?style=for-the-badge)](LICENSE)

[English](README.en.md) · [中文](README.zh.md) · [SPEC.md](SPEC.md) · [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension)

</div>

---

```bash
$ cat home.kx
```

```kx
@page /home (首页) extends AppLayout {
  @slot main {
    @list 作品流 {
      @api GET /works (query: page, tab) -> works

      @card 作品卡片 (v-for: item in works) {
        @prop item: Work
        @media 封面 (bind: item.cover_url)
        @text 标题 (bind: item.title)

        @button 点赞 (bind: item.likes_count) {
          @api POST /works/:id/like
          @mutation set works.find(w => w.id === id).likes_count += 1
          @login
        }

        @hover -> @popover 详情预览
      }
    }
  }
}
```

<div align="center">

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;☝️&nbsp;&nbsp;一段 `.kx` 描述&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;👇&nbsp;&nbsp;AI 自动生成&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

</div>

```bash
$ ls src/
  api/works.ts          ← @api 自动封装
  router/index.ts       ← @page 自动注册
  views/HomeView.vue    ← @list/@card 模板
  components/cards/
    PoseCard.vue         ← @prop/@media/@button 子组件
  stores/interaction.ts ← @mutation 状态管理
```

<table>
<tr></tr>
<tr>
<td>

**你写架构描述，AI 写代码。**

`.kx` 文件是页面的单一事实源——结构、数据流、交互、权限、业务规则，汇聚于一处。不再零散提示词，不再反复调试。

</td>
<td>

**You write the blueprint. AI writes the code.**

A `.kx` file is the single source of truth for a page — structure, data flow, interactions, permissions, and business rules in one place. No more scattered prompts, no more guesswork.

</td>
</tr>
</table>

---

## ⚡ Three Steps / 三步上手

<table>
<tr></tr>
<tr>
<td width="33%">

### 1. 描述页面
用 `@page` `@list` `@card` 等 23 种语义指令描述页面结构
</td>
<td width="33%">

### 2. 投喂给 AI
将 `.kx` 文件 + `SPEC.md` 放入 Cursor / Claude 上下文
</td>
<td width="33%">

### 3. 生成代码
AI 输出完整 Vue 3 工程：组件、API、路由、Store
</td>
</tr>
</table>

---

## 📊 Before vs After / 对比

<table>
<tr></tr>
<tr>
<th width="50%">❌ 传统方式</th>
<th width="50%">✅ KX 方式</th>
</tr>
<tr>
<td>

```
"帮我写一个作品瀑布流页面，
要有分页、搜索、点赞功能，
点赞要乐观更新..."

→ AI 猜测结构 → 一次次的调试 → 细节遗漏
```

</td>
<td>

```kx
@card 作品卡片 (v-for: item in works) {
  @button 点赞 {
    @api POST /works/:id/like
    @mutation set works.find(w => w.id === id).liked = true
    @login
  }
}

→ 精确 → 一次成功 → 细节完整
```

</td>
</tr>
</table>

---

## ✨ 核心特性 / Features

<table>
<tr></tr>
<tr>
<td width="50%">

📐 **块级语义**
23 种指令覆盖布局、内容、交互、反馈四大类

🔗 **数据驱动**
`@api` + `@mutation` + `@sync` 让数据流清晰可见

🎨 **交互一等公民**
`@hover` 双模式、`@popover`、`@position`、`@delay`

🔒 **安全内建**
`@login` 三模式守卫 + `@permission` 权限控制

🤖 **AI 原生**
`@note` 让业务约束对 AI 强制执行

</td>
<td width="50%">

📐 **Block-level Semantics**
23 directives across layout, content, interaction, feedback

🔗 **Data-driven**
`@api` + `@mutation` + `@sync` for visible data flow

🎨 **Interaction First**
`@hover` dual mode, `@popover`, `@position`, `@delay`

🔒 **Secure by Default**
`@login` three-mode guard + `@permission` control

🤖 **AI Native**
`@note` enforces business constraints on AI

</td>
</tr>
</table>

---

## 🛠️ 技术栈 / Tech Stack

```
KX Specification     → SPEC.md (this repo)
VS Code Highlighting → kx-lang-extension
Real-world Usage     → PoseCraft (production example)
```

---

## 📖 Documentation / 文档

| 文件 | 说明 | Description |
|:---|:---|:---|
| [SPEC.md](SPEC.md) | 完整语法规范 + AI 生成映射表 | Full spec + AI mapping table |
| [example.kx](example.kx) | 4 个实战场景示例 | 4 real-world examples |
| [README.zh.md](README.zh.md) | 中文完整文档 | Chinese complete docs |
| [README.en.md](README.en.md) | English complete docs | English complete docs |

---

## 🔗 关联仓库 / Related Repos

| 仓库 | 说明 |
|:---|:---|
| [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension) | VS Code 语法高亮扩展 |
| [PoseCraft](https://github.com/yijiu2025/CoreFlow/tree/main/posecraft) | KX 在大型项目中的生产级应用 |

---

## 📋 路线图 / Roadmap

<table>
<tr></tr>
<tr>
<td>

**已完成**
- [x] v1.0 — 基础指令
- [x] v1.1 — 数据流 + 状态
- [x] v1.2 — 悬浮交互 + 权限 + @note

</td>
<td>

**进行中**
- [ ] v2.0 — 服务端函数 `@server`、WebSocket `@ws`
- [ ] v2.1 — 多端适配、AI 生成器集成

</td>
</tr>
</table>

---

## 🤝 参与贡献 / Contributing

欢迎任何形式的贡献：

- 📝 完善 KX 规范或新增示例
- 🎨 增强 `kx-lang-extension` 语法支持
- 📖 优化文档与写作指南
- 🐛 提交 Issue 或 PR

---

<div align="center">

## 🚀 现在开始 / Get Started

```bash
# 1. 安装语法高亮
code --install-extension kx-lang-extension/kx-lang-extension-0.0.1.vsix

# 2. 阅读规范
open SPEC.md

# 3. 开始编写你的第一个 .kx 文件
touch my-page.kx
```

<sub>如果觉得有用，请给个 ⭐ 支持一下！ · If this helps, a ⭐ is appreciated!</sub>

<br/>

[MIT](LICENSE) · Copyright (c) 2025 KX Language Contributors

</div>
