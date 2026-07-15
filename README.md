<div align="center">

# KX — 页面描述语言

**K**nowledge e**X**change · 让 AI 读懂业务，让代码生于架构

<!-- prettier-ignore -->
[![Version](https://img.shields.io/badge/v1.2-5B6DEF?style=for-the-badge&logo=bookmark&logoColor=white)](https://github.com/yijiu2025/kx-lang/releases)
[![AI Native](https://img.shields.io/badge/AI-原生-00D26A?style=for-the-badge&logo=openai&logoColor=white)](SPEC.md)
[![Vue 3](https://img.shields.io/badge/目标-Vue_3-42b883?style=for-the-badge&logo=vue.js&logoColor=white)](https://vuejs.org/)
[![License](https://img.shields.io/badge/协议-MIT-E12B89?style=for-the-badge)](LICENSE)

[📖 English](README.en.md) · [📘 规范](SPEC.md) · [🎨 语法高亮](https://github.com/yijiu2025/kx-lang-extension)

</div>

---

## 一、KX 是什么？

**KX** 是一种专为 **AI 代码生成** 设计的声明式页面描述语言。

用 `.kx` 文件描述页面的结构、数据流、交互逻辑和业务规则，AI 就能自动生成完整、类型安全、生产就绪的 Vue 3 代码。

```kx
@page /home (首页) extends AppLayout {
  @slot main {
    @list 作品流 {
      @api GET /works (query: page, tab) -> works

      @card 作品卡片 (v-for: item in works) {
        @prop item: Work
        @media 封面 (bind: item.cover_url)
        @text 标题 (bind: item.title)

        @button 点赞 {
          @api POST /works/:id/like
          @mutation set works.find(w => w.id === id).likes_count += 1
          @login
        }
      }
    }
  }
}
```

**你写架构描述，AI 写代码。** 不再需要零散的提示词和反复调试。

---

## 二、为什么选择 KX？

### 🎯 业务优先，AI 友好

传统方式：你用自然语言描述需求 → AI 猜测结构 → 反复调试 → 细节遗漏

KX 方式：你用结构化指令声明意图 → AI 精确解析 → **一次成功 → 细节完整**

### 📊 对比一览

| | ❌ 传统方式 | ✅ KX 方式 |
|:---|:---|:---|
| **描述方式** | 自然语言提示词 | 结构化指令 |
| **AI 理解** | 猜测意图 | 精确解析 |
| **产出质量** | 反复调试 | 一次成功 |
| **业务逻辑** | 散落在组件中 | 集中在 `@note` |
| **数据流** | 隐式，需要推断 | 显式，`@api` + `@mutation` |
| **维护方式** | 逐文件 code review | 改描述 → 重新生成 |

### ✨ 核心优势

| | 优势 | 说明 |
|:---:|:---|:---|
| 📐 | **块级语义** | 23 种指令覆盖布局、内容、交互、反馈四大类 |
| 🔗 | **数据驱动** | `@api` + `@mutation` + `@sync`，数据流清晰可见 |
| 🎨 | **交互一等公民** | `@hover` 双模式、`@popover`、`@delay`，复杂交互一行搞定 |
| 🔒 | **安全内建** | `@login` 三模式守卫 + `@permission` 权限控制 |
| 🤖 | **AI 原生** | `@note` 让业务约束对 AI **强制执行**，不再是可忽略的注释 |

---

## 三、核心概念

### 三类指令

| 类型 | 指令 | 作用 |
|:---|:---|:---|
| **结构指令** | `@page` `@layout` `@slot` | 定义页面骨架和区域 |
| **组件指令** | `@button` `@list` `@card` `@tab` `@header` `@modal` ... | 23 种，覆盖所有 UI 场景 |
| **逻辑指令** | `@api` `@mutation` `@navigate` `@render` `@login` `@note` | 数据、交互、权限、业务规则 |

### 一个例子看懂 KX

<table>
<tr></tr>
<tr>
<th width="40%">KX 输入</th>
<th width="60%">AI 输出</th>
</tr>
<tr>
<td>

```kx
@card 作品卡片 (v-for: item in works) {
  @prop item: Work
  @media 封面 (bind: item.cover_url)
  @text 标题 (bind: item.title)

  @button 点赞 {
    @api POST /works/:id/like
    @mutation set works.find(w => w.id === id).likes_count += 1
    @login
  }
}
```

</td>
<td>

```vue
<!-- components/cards/PoseCard.vue -->
<template>
  <div class="card" @click="openDetail">
    <img :src="item.cover_url" />
    <span>{{ item.title }}</span>
    <button @click.stop="handleLike">
      ❤️ {{ item.likes_count }}
    </button>
  </div>
</template>
<script setup>
defineProps<{ item: Work }>()
async function handleLike() {
  if (!user.isLogin) return router.push('/login')
  await worksApi.like(item.id)
  // 乐观更新
  const idx = works.value.findIndex(w => w.id === item.id)
  works.value[idx].likes_count++
}
</script>
```

</td>
</tr>
</table>

---

## 四、快速上手

### 第一步：安装语法高亮（可选）

```bash
# 方式一：本地安装
code --install-extension kx-lang-extension/kx-lang-extension-0.0.1.vsix

# 方式二：VS Code 市场搜索 "KX Syntax Highlight"
```

### 第二步：阅读规范

- [SPEC.md](SPEC.md) — 完整语法规范 + AI 生成映射表（30+ 条映射规则）

### 第三步：编写你的第一个 `.kx` 文件

```
项目结构：
├── layouts/app.kx          ← 全局布局
├── pages/
│   ├── home.kx             ← 首页
│   ├── mine.kx             ← 个人中心
│   └── detail.kx           ← 详情页
└── components/
    └── shared.kx           ← 公共组件定义
```

### 第四步：投喂给 AI

将 `.kx` 文件 + `SPEC.md` 放入 Cursor / Claude 上下文，然后说：

> "请严格遵循 SPEC.md 中的 KX 规范。读取 .kx 文件作为唯一架构来源，将其转换为 Vue 3 + TypeScript 代码。"

---

## 五、实战案例

### 案例 1：精选首页

```kx
@page / (精选) extends AppLayout {
  @icon HomeOutlined
  @meta title="精选" description="发现最新作品"

  @slot main {
    @tab 频道 {
      @api GET /config/channels -> channels
      @state active: string = 'recommend'
    }

    @banner 轮播 {
      @api GET /banner-configs/active -> banners
      @render when: active === 'recommend' && banners.length
    }

    @list 作品流 {
      @api GET /works (query: page, channel: active) -> works
      @loading { @skeleton 卡片 (count: 8) }

      @empty {
        @empty 空状态 {
          @text 暂无作品
          @button 去投稿 { @navigate click -> /editor }
        }
      }

      @card 作品卡片 (v-for: item in works) {
        @prop item: Work
        @media 封面 (bind: item.cover_url)
        @text 标题 (bind: item.title)
        @badge 热门 { @render when: item.hot_score > 80 }

        @action 操作栏 {
          @button 点赞 (bind: item.likes_count) {
            @api POST /works/:id/like
            @mutation set works.find(w => w.id === id).likes_count += 1
            @login
          }
          @button 收藏 {
            @api POST /interaction/collect @login
          }
        }
      }

      @button 加载更多 {
        @api GET /works (page: nextPage) -> works(append)
        @render when: hasMore
      }
    }
  }
}
```

AI 产出：`HomeView.vue` + `PoseCard.vue` + `api/works.ts` + `router/index.ts` + `stores/works.ts`

---

### 案例 2：作品详情（含高级悬浮交互）

```kx
@page /work/:id (作品详情) extends AppLayout {
  @param id: string { from: route }

  @slot main {
    @detail 作品详情 {
      @api GET /works/:id (bind: id) -> work

      @media 主图 (bind: work.image_url)

      @author 作者卡 {
        @avatar 头像 (bind: work.author.avatar) {
          @hover -> @popover 作者预览
          @navigate click -> /user/:id
        }
        @button 关注 {
          @api POST /follow/:userId (bind: work.author.id)
          @login
        }
      }

      @text 标题 (bind: work.title)
      @text 描述 (bind: work.description) { @note 超出折叠 }

      @action 互动栏 {
        @button 点赞 (bind: work.likes_count) {
          @api POST /works/:id/like
          @mutation set work.likes_count += 1
          @login
        }
        @button 收藏 { @api POST /interaction/collect @login }
        @button 评论 { @event click -> scrollToComments }
        @button 分享
      }

      @button 使用此姿势拍照 {
        @render when: work.template_id
        @navigate click -> /camera?template_id=:id
      }
    }
  }
}
```

---

### 案例 3：个人中心（含 @hover 手动模式 + 批量管理）

```kx
@page /mine (我的空间) extends AppLayout {
  @login
  @note 未登录跳转登录页，登录后回跳

  @slot main {
    @header 个人资料 {
      @api GET /profile/stats -> stats
      @avatar 头像 (bind: stats.avatar)
      @stat 关注 (bind: stats.following) { @navigate click -> /following }
      @stat 粉丝 (bind: stats.followers) { @navigate click -> /followers }
      @button 编辑资料 { @permission self }
    }

    @tab 分类 {
      @state activeTab: string = 'works'
      @menu 作品 { @sync = worksCount }
      @menu 模板 { @sync = templatesCount }
      @menu 喜欢 / 收藏 / 历史
    }

    @list 作品列表 {
      @render when: activeTab === 'works'
      @api GET /works/mine -> myWorks

      @card 作品卡片 (v-for: item in myWorks) {
        // 模式二：手动控制 + 延迟 + 锚点定位
        @state showDetail = false
        @hover { @state showDetail = true @delay show: 300ms }
        @leave { @delay hide: 200ms }

        @media 封面 (bind: item.cover)

        @popover 详情预览 {
          @render when: showDetail
          @anchor @card 作品卡片
          @position right-top
          @api GET /works/:id/detail (bind: item.id) -> detail
          @media 大图 (bind: detail.hd_image)
          @button 快速点赞 {
            @api POST /works/:id/like
            @mutation set myWorks.find(w => w.id === id).likes_count += 1
          }
        }

        @button 删除 {
          @api DELETE /works/:id
          @mutation filter myWorks = myWorks.filter(w => w.id !== id)
        }
      }
    }
  }
}
```

更多示例见 [example.kx](example.kx)（4 个完整场景）。

---

## 六、语法速览

### 结构指令

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@page` | 页面路由 | `@page /home (首页)` |
| `@layout` | 全局布局 | `@layout AppLayout { ... }` |
| `@slot` | 区域/插槽 | `@slot main (role: main)` |
| `@icon` | 页面图标 | `@icon HomeOutlined` |
| `@meta` | SEO 元信息 | `@meta title="首页"` |
| `@param` | 路由参数 | `@param id: string { from: route }` |

### 组件指令（23 种）

| 类别 | 指令 |
|:---|:---|
| 布局 | `@header` `@sidebar` `@banner` `@detail` `@logo` |
| 内容 | `@card` `@list` `@text` `@media` `@badge` `@avatar` `@author` `@stat` |
| 交互 | `@button` `@input` `@tab` `@menu` `@action` `@form` |
| 反馈 | `@modal` `@popover` `@toast` `@empty` `@skeleton` |

### 逻辑指令

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@api` | 接口请求 | `@api GET /works -> list` |
| `@mutation` | 状态变更 | `@mutation set list[idx].liked = true` |
| `@sync` | 计算属性 | `@sync = myWorks.length` |
| `@state` | 本地状态 | `@state loading: boolean = false` |
| `@prop` | 组件属性 | `@prop item: Work` |
| `@navigate` | 路由跳转 | `@navigate click -> /detail` |
| `@render` | 条件渲染 | `@render when: loading` |
| `@login` | 登录守卫 | `@login` |
| `@permission` | 权限控制 | `@permission work:create` |
| `@note` | 业务说明 | `@note 仅 VIP 可见` |

### 悬浮交互指令

| 语法 | 作用 | 示例 |
|:---|:---|:---|
| `@hover -> @popover` | 自动托管模式 | `@hover -> @popover 详情` |
| `@hover { ... }` | 手动控制模式 | `@hover { @state s = true @delay show: 300ms }` |
| `@leave { ... }` | 鼠标离开 | `@leave { @delay hide: 200ms }` |
| `@anchor` | 浮窗锚点 | `@anchor @card 作品卡片` |
| `@position` | 浮窗方位 | `@position bottom-left` |
| `@delay` | 延迟显示/隐藏 | `@delay show: 200ms` |

---

## 七、路线图

| 版本 | 状态 | 内容 |
|:---|:---|:---|
| v1.0 | ✅ | 基础指令：`@page` `@slot` `@button` `@api` `@navigate` |
| v1.1 | ✅ | 数据流：`@mutation` `@sync` `@render` `@state` `@prop` `@param` |
| v1.2 | ✅ | 交互+权限：`@hover` `@leave` `@anchor` `@position` `@delay` `@login` `@note`；23 种组件 |
| v2.0 | 🔜 | 服务端函数 `@server`、WebSocket `@ws`、自定义指令扩展 |
| v2.1 | 📋 | 多端适配 `@platform`、AI 生成器集成 |

---

## 八、关联仓库

| 仓库 | 说明 |
|:---|:---|
| [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension) | VS Code 语法高亮扩展 |
| [PoseCraft](https://github.com/yijiu2025/CoreFlow/tree/main/posecraft) | KX 在大型项目中的生产级应用（8 个 .kx 文件，2100+ 行） |

---

## 九、参与贡献

欢迎任何形式的贡献：

- 📝 完善 KX 规范或新增 `.kx` 示例
- 🎨 增强 `kx-lang-extension` 语法支持
- 📖 优化文档与写作指南
- 🐛 提交 Issue 或 Pull Request

---

<div align="center">

[📖 中文](README.md) · [English](README.en.md) · [📘 规范](SPEC.md)

[MIT](LICENSE) · Copyright (c) 2025 KX Language Contributors

<sub>如果觉得有用，请给个 ⭐ 支持一下！</sub>

</div>
