<div align="center">

# KX — 页面描述语言

**K**nowledge e**X**change · 让 AI 读懂业务，让代码生于架构

[![Version](https://img.shields.io/badge/version-1.2-blue?style=flat-square)]()
[![Language](https://img.shields.io/badge/DSL-Declarative-orange?style=flat-square)]()
[![AI Native](https://img.shields.io/badge/AI-Native-brightgreen?style=flat-square)]()
[![Target](https://img.shields.io/badge/target-Vue_3-42b883?style=flat-square)](https://vuejs.org/)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)]()

[文档](SPEC.md) · [示例](example.kx) · [扩展](kx-lang-extension) · [贡献](#参与贡献)

</div>

---

## 这是什么？

**KX** 是一种专为 AI 驱动前端构建设计的声明式页面描述语言。

用 `.kx` 文件描述页面结构、数据流、交互逻辑、权限和业务规则，AI 可以自动生成 Vue 3 代码。

---

## 你能用它做什么

- 用一份 `.kx` 文件表达页面布局和互动。
- 用 `@api` 定义接口，用 `@mutation` 明确状态更新。
- 用 `@note` 写业务约束，让模型精准理解。
- 用 `@login` / `@permission` 定义访问控制。
- 用 `@hover` / `@popover` 描述复杂悬浮交互。

---

## 快速上手

```kx
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

---

## 写作规则

### 1. 结构清晰

- 使用 `@<类型> <名称> { ... }` 进行块级组织。
- 每个 `.kx` 文件应聚焦一个页面或一个独立模块。
- 避免将多个页面写在同一个文件中。

### 2. 业务说明要用 `@note`

`@note` 是 AI 识别业务约束的关键：

- `@note 仅 VIP 用户可见`
- `@note 点击后上报埋点 submit_click`
- `@note 搜索时支持联想提示`

### 3. 数据与状态显式

- `@prop item: Work`
- `@state page: number = 1`
- `@param id: string { from: route }`
- `@api GET /path -> state`
- `@mutation` 说明本地变更

### 4. 交互要明确

- `@navigate click -> /detail`
- `@render when: loading`
- `@hover -> @popover`
- `@hover { @state show = true @delay show: 200ms }`
- `@leave { @state show = false }`

### 5. 与扩展语法保持一致

本仓库的 `kx-lang-extension` 已支持以下指令：

- `@page`, `@layout`, `@slot`
- `@api`, `@navigate`, `@render`, `@permission`, `@login`
- `@state`, `@prop`, `@param`, `@mutation`, `@event`
- `@hover`, `@popover`, `@anchor`, `@position`
- `@note`, `@toast`, `@empty`, `@skeleton`

---

## 项目结构

```
.
├── SPEC.md
├── README.md
├── app.kx
├── example.kx
└── kx-lang-extension/
    ├── package.json
    ├── syntaxes/kx.tmLanguage.json
    └── ...
```

---

## 扩展同步说明

`kx-lang-extension` 语法高亮已覆盖 `.kx` 关键语法，可直接用于编写和阅读页面架构。

---

## 发布扩展

1. 设置 `kx-lang-extension/package.json` 中的 `publisher`
2. 安装发布工具：

```bash
npm install -g @vscode/vsce
```

3. 打包并发布：

```bash
cd kx-lang-extension
npx @vscode/vsce package
npx @vscode/vsce publish
```

> 发布前请确认 `SPEC.md`、`README.md`、`app.kx`、`example.kx` 已完成同步。

---

## 参与贡献

欢迎提交 Issues 或 PR：

- 扩展新的 `@` 指令
- 丰富 `.kx` 示例
- 优化语法高亮、配色和文档
- 细化写作规则与最佳实践

---
