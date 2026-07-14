# KX - 页面描述语言 (v1.1)

> **K**nowledge e**X**change · 让 AI 读懂业务，让代码生于架构

[![Version](https://img.shields.io/badge/version-1.1-blue.svg)](https://github.com/yourname/kx-lang)
[![AI Native](https://img.shields.io/badge/AI-Native-brightgreen.svg)](https://github.com/yourname/kx-lang)
[![Vue 3](https://img.shields.io/badge/Target-Vue_3-42b883.svg)](https://vuejs.org/)

---

## 📖 简介

**KX** 是一种专为 **AI 代码生成** 设计的声明式页面描述语言。

它既不是新的 JavaScript 框架，也不是臃肿的低代码平台，而是一套 **“可执行的架构蓝图”**。你可以像写产品大纲一样描述页面，AI（如 Cursor / Claude）则能根据这些描述，自动生成类型安全、生产就绪的 Vue 3 / React 代码。

> **核心理念**：让业务逻辑回归人类可读的文本，把重复的编码工作交给 AI。

---

## ✨ 核心特性

-   **🧠 人类可读，AI 可解析**：语法极简，产品经理能看懂，AI 更能精准映射为组件代码。
-   **🧩 布局继承 (DRY)**：通过 `@layout` 定义全局骨架，子页面只需关心核心内容，彻底消灭重复代码。
-   **📝 注释即文档，注释即逻辑**：支持自然语言注释，AI 生成代码时会自动将注释转化为业务逻辑（如 `// 仅推荐频道展示` → `v-if`）。
-   **🏷️ 页面元数据原生支持**：内置 `@icon` 和 `@meta`，AI 生成路由时自动注册标题和图标。
-   **⚡ 智能数据流管理**：通过 `@api` + `@mutation` 实现乐观更新（Optimistic Update），代码量减少 70%。
-   **🎯 显式状态管理**：通过 `@state`、`@prop` 和 `@param` 明确数据来源（路由参数/组件属性），杜绝类型 `any`。

---

## 🚀 为什么需要 KX？

| 传统开发流程 | KX + AI 开发流程 |
| :--- | :--- |
| 画原型图 → 写 Vue 模板 → 写 Script 逻辑 → 调接口 → 改 Bug | **画架构图 (KX)** → **拖拽给 AI** → **直接运行** |
| 修改需求需要改 `.vue`、`.ts`、`.scss` 三个文件 | 只需修改 `.kx` 文件中的描述，AI 自动同步更新所有文件 |
| 组件间数据流动晦涩，新人上手慢 | `@api` 和 `@mutation` 清晰标注了数据的流入流出 |

---

## 📚 语法速览

这是一个简洁的快速参考，完整规范请见 [SPEC.md](./SPEC.md)。

### 1. 页面与布局

```kx
// 定义全局骨架（只需写一次）
@layout MainLayout {
  @slot top (role: header) { ... 顶栏 ... }
  @slot left (role: aside) { ... 侧边栏 ... }
  @slot main (role: main) { /* 留给子页面填充 */ }
}

// 子页面继承布局，自带路由和图标
@page /home (首页) extends MainLayout {
  @icon HomeOutlined
  @meta title="首页"
  
  @slot main {
    // 只关心核心内容
  }
}
```
### 2. 组件与数据
```kx
@component list Waterfall {
  // 请求数据，参数绑定本地状态
  @api GET /api/works (query: page, channel: activeTab) -> works

  // 处理加载和空状态
  @loading { @component skeleton SkeletonCard }
  @empty { @component empty EmptyState }

  // 遍历渲染
  @component card ProductCard (v-for: item in works) {
    @prop item: Product
    
    // 点赞：乐观更新（先改 UI，失败回滚）
    @action 点赞 {
      @api POST /api/works/:id/like
      @mutation set works.find(w => w.id === id).liked = true
      @mutation set works.find(w => w.id === id).likes_count += 1
    }
  }
}
```
### 3. 权限与导航
```kx
@component button 投稿 {
  @navigate click -> /editor
  @permission posecraft:work:create  // 无权限自动隐藏
}

@component avatar 头像 @navigate click -> /mine @login // 未登录跳转登录页
```
##🤖 AI 生成映射表 (输入 → 输出)
KX 写法	AI 生成的 Vue 3 代码
@api GET /works -> list	const { data: list } = await api.getWorks()
@state loading: boolean	const loading = ref(false)
@mutation set list[idx].liked = true	list.value[index].liked = true
@render when: loading	v-if="loading"
@navigate click -> /detail	@click="router.push('/detail')"
@slot top (role: header)	<header slot="top">

🛠️ 如何与 AI 配合使用（最佳实践）
要让 KX 发挥最大威力，请按以下步骤操作（以 Cursor 或 VSCode + Copilot 为例）：

投喂规范：将本仓库的 SPEC.md 放在项目根目录，并在 .cursorrules 或系统提示词中加入：

“请严格遵循 SPEC.md 中的 KX 规范。读取 .kx 文件作为唯一架构来源，将其转换为 Vue 3 + TypeScript 代码。遇到 @api 自动生成对应的 api/service 文件。”

编写架构：在 pages/ 目录下编写 .kx 文件。多用注释！你写的中文注释（如 // 只有VIP可见），AI 会精准翻译为对应的 v-if 条件。

生成代码：在 Cursor 的 Composer 中 @ 引用你的 .kx 文件，输入指令：“根据这个文件生成完整的页面代码”。

迭代维护：需求变更时，只修改 .kx 文件中的描述（比如把 list 改为 grid），然后让 AI 同步修改组件。

📜 规范详情
完整的语法定义、关键词列表和高级用法（如 @typedef 类型定义），请查阅项目根目录下的 SPEC.md。

🤝 贡献
KX 还处于早期孵化阶段。我们欢迎：

对语法的建议（如何让 AI 理解更精准）

更多前端框架（React/Svelte）的映射模板

配套的 VSCode 语法高亮插件

请提交 Issue 或 Pull Request。

