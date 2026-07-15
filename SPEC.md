# KX 页面描述语言规范 v1.2

> **一种让 AI 读懂页面架构、自动生成 Vue 3 代码的声明式 DSL。**
> 结构化语法、零歧义、从骨架到细节逐层展开。

## 设计原则

1. **人类可读**：像写大纲一样描述页面，层级即语义。
2. **AI 可解析**：`@<类型> <名称> {}` 块式结构，无歧义。
3. **代码可生成**：一句描述即可输出组件、接口、路由。
4. **渐进增强**：从布局骨架到交互细节逐层叠加。
5. **块级语义**：所有结构均用 `@` 指令明确表达。

---

## 一、语法结构

### 1. 注释与业务说明

```kx
// 单行注释（编译时忽略）
/* 多行注释 */
@note 显式业务规则，AI 必须参考
```

`@note` 与普通注释不同——它是**业务约束声明**，生成器必须将其视为实现依据。

### 2. 页面与布局

```kx
@page /path (标题) extends AppLayout {
  @icon HomeOutlined
  @meta title="..." description="..."
  @param id: string { from: route }
  @slot main { ... }
}

@layout 布局名 {
  @slot left (role: aside) { ... }
  @slot top (role: header) { ... }
  @slot main (role: main) { ... }
}
```

| 元素 | 说明 |
|:---|:---|
| `@page` | 页面定义，path 为路由路径，括号内为中文标题 |
| `@icon` | 页面/菜单图标（Ant Design / Arco 图标名） |
| `@meta` | SEO meta 信息 |
| `extends` | 继承布局，子页面填充对应 @slot |
| `@layout` | 文件级布局定义，可跨页面复用 |
| `@slot` | 插槽占位，声明 role 标识区域类型 |

### 3. 数据声明

```kx
@state page: number = 1               // 页面级响应式状态
@prop item: Work                       // 组件 prop 类型声明
@param id: string { from: route }     // 路由参数（route / query）
@state activeTool: string = 'select'  // 带默认值
@param template_id: string { from: query }
```

- `@state`：声明响应式变量，支持 `string | number | boolean | object | array` 类型。
- `@prop`：声明组件接收的 props（用于循环内子组件）。
- `@param`：声明路由参数来源（`from: route` 路径参数，`from: query` 查询参数）。

### 4. 数据交互

#### @api — 接口请求

```kx
@api GET /path (query: page, keyword) -> works
@api POST /path (body: { id, title })
@api PUT /works/:id (bind: id)
@api GET /works/:id (query: camera=true) -> works(append)
```

| 语法 | 说明 |
|:---|:---|
| `GET/POST/PUT/DELETE/PATCH/HEAD/OPTIONS` | HTTP 方法 |
| `/works/:id` | 路径参数 |
| `(query: a, b)` | 查询参数列表 |
| `(body: { ... })` | 请求体 |
| `(bind: var)` | 绑定变量到路径 `:param` |
| `(formData: file)` | 文件上传参数 |
| `-> stateName` | 结果写入状态 |
| `-> stateName(append)` | 追加到数组（加载更多） |

#### @mutation — 本地状态变更

```kx
@mutation set user.username = editForm.username
@mutation set works.find(w => w.id === id).likes_count += 1
@mutation set works.find(w => w.id === id).liked = true
@mutation filter works = works.filter(w => w.id !== item.id)
@mutation remove works.filter(w => w.author.id === id)
```

| 动词 | 说明 |
|:---|:---|
| `set` | 赋值或更新属性（支持 `.find()` 定位） |
| `filter` | 过滤数组 |
| `remove` | 移除元素（语法糖，等价 filter） |

#### @sync — 计算属性

```kx
@sync isVip = user.level >= 5
@sync filteredItems = works.filter(w => w.title.includes(keyword))
```

### 5. 组件类型（23 种）

#### 布局组件

| 指令 | 说明 | 典型用途 |
|:---|:---|:---|
| `@header` | 顶部栏 | 导航、工具栏 |
| `@sidebar` | 侧边栏 | 左侧菜单、工具面板 |
| `@banner` | 横幅区 | 首页轮播、活动推广 |
| `@detail` | 详情容器 | 作品详情、编辑器主体 |
| `@social` | 社交信息流 | 右侧评论区、元数据 |
| `@logo` | 品牌 Logo | 侧边栏顶部、登录页 |

#### 内容组件

| 指令 | 说明 | 典型用途 |
|:---|:---|:---|
| `@card` | 卡片 | 作品卡片、评论项 |
| `@list` | 列表容器 | 瀑布流、评论列表、动态流 |
| `@text` | 文本 | 标题、描述、统计值 |
| `@media` | 图片/视频 | 封面、视频、相机取景 |
| `@badge` | 角标 | 状态标签、距离标签 |
| `@avatar` | 头像 | 用户头像、作者头像 |
| `@author` | 作者信息卡 | 含头像、统计、关注按钮 |
| `@stat` | 统计数字 | 粉丝数、作品数 |

#### 交互组件

| 指令 | 说明 | 典型用途 |
|:---|:---|:---|
| `@button` | 按钮 | 提交、导航、操作触发 |
| `@input` | 输入框 | 搜索、表单、滑块 |
| `@tab` | 选项卡 | 频道切换、面板切换 |
| `@menu` | 菜单 | 导航菜单、下拉菜单 |
| `@action` | 操作栏 | 底部工具栏、互动栏 |
| `@form` | 表单容器 | 编辑表单、注册表单 |

#### 反馈组件

| 指令 | 说明 | 典型用途 |
|:---|:---|:---|
| `@modal` | 弹窗 | 确认框、表单弹窗、帮助 |
| `@popover` | 悬浮卡 | 头像卡、预览卡、颜色选择器 |
| `@toast` | 轻提示 | 操作反馈、错误提示 |
| `@empty` | 空状态 | 无数据占位 |
| `@skeleton` | 骨架屏 | 加载占位（`count: 8`） |

#### 循环与属性绑定

```kx
@card 卡片 (v-for: item in works) { ... }
@media 封面 (bind: item.cover_url)
@text 标题 (bind: item.title)
@stat 粉丝 (bind: user.followersCount)
```

### 6. 条件渲染与循环

```kx
@render when: works.length > 0
@render when: item.status === 'published'
@render when: isVip
@render unless: !works.length
@render when: activeTool === 'draw'
@render when: showOverlay && work.image_url
@render when: !hasMore && works.length > 0
```

- `when:` — 条件为真时渲染。
- `unless:` — 条件为假时渲染。
- 支持比较、逻辑运算符（`&&` `||` `===` `!==` `>` `<`）。

### 7. 事件与导航

#### @navigate — 路由跳转

```kx
@navigate click -> /editor?id={item.id}
@navigate click -> /work/:id
@navigate click -> @modal 删除确认
@navigate click -> @popover 颜色选择器
@navigate success -> /
@navigate click -> back
@navigate click -> @action saveData
@navigate prefetch -> /work/:id
@navigate click -> @state show = true
```

| 目标类型 | 示例 | 说明 |
|:---|:---|:---|
| 路径 | `-> /work/:id` | 路径参数自动替换 |
| 路径+查询 | `-> /editor?id={item.id}` | 查询拼接 |
| @action | `-> @action saveData` | 触发 action |
| @popover | `-> @popover 颜色选择器` | 展开浮窗 |
| @modal | `-> @modal 删除确认` | 打开弹窗 |
| @prefetch | `-> @prefetch` | 预加载路由 |
| @state | `-> @state show = true` | 切换状态 |
| back | `-> back` | 浏览器后退 |
| success | `-> success` | 成功后导航 |

#### @event — DOM 事件

```kx
@event click -> @api retry
@event submit -> @mutation set submitted = true
```

### 8. 悬浮交互

#### 自动托管模式

```kx
@avatar 头像 {
  @hover -> @popover 头像悬浮卡
}
```

#### 手动控制模式

```kx
@hover {
  @state show = true
  @delay show: 300ms
}
@leave {
  @delay hide: 200ms
}
@popover 详情浮窗 {
  @render when: show
  @anchor @card 作品卡片
  @position bottom
}
```

| 指令 | 说明 |
|:---|:---|
| `@hover` | 悬停触发容器 |
| `@leave` | 离开触发容器 |
| `@delay` | 延迟显示/隐藏（格式化：`300ms`） |
| `@anchor` | 浮窗锚定目标组件 |
| `@position` | 浮窗位置（top/bottom/left/right） |

### 9. 权限与守卫

```kx
@button 投稿 {
  @permission posecraft:work:create
  @login
}
```

提供三种模式：

| 模式 | 写法 | 行为 |
|:---|:---|:---|
| 仅登录 | `@login` | 未登录跳转登录页 |
| 仅权限 | `@permission code` | 无权限隐藏元素 |
| 组合 | 两者叠加 | 先校验登录，再校验权限 |

### 10. 状态容器

```kx
@list 作品列表 {
  @loading {
    @skeleton 卡片骨架 (count: 8)
  }
  @empty {
    @empty 空状态 {
      @icon 📂
      @text 暂无内容
      @button 立即投稿 { @navigate click -> /editor }
    }
  }
  @error {
    @toast "加载失败，点击重试"
    @event click -> @api retry
  }
  @card 作品卡片 (v-for: item in works) { ... }
}
```

`@loading` / `@empty` / `@error` 是 `@list` 内部的生命周期容器，分别对应加载中、无数据、接口异常三种状态。

---

## 二、完整实战示例

### 示例 1：全局布局（app.kx）

```kx
@layout PosecraftLayout {
  @slot left (role: aside) {
    @sidebar 侧边栏 {
      @logo Logo { @navigate click -> / }
      @menu 发现 {
        @menu 精选 @navigate click -> /
        @menu 推荐 @navigate click -> /recommend
      }
      @menu 社交 {
        @menu 我的 @navigate click -> /mine @login
      }
    }
  }
  @slot top (role: header) {
    @header 顶部栏 {
      @input 搜索框 {
        @api GET /works (query: keyword) -> results
      }
      @avatar 头像 {
        @hover -> @popover 头像悬浮卡
        @navigate click -> /mine @login
      }
    }
  }
  @slot main (role: main) { }
}

@popover 头像悬浮卡 {
  @avatar 头像 (bind: user.avatar)
  @text 用户名 (bind: user.username)
  @stat 粉丝 (bind: user.followersCount)
  @menu 退出登录 {
    @api POST /auth/logout
    @navigate success -> /login
  }
}
```

### 示例 2：首页频道页

```kx
@page / (精选) extends PosecraftLayout {
  @icon HomeOutlined
  @meta title="精选姿势"

  @slot main {
    @tab 频道Tab {
      @api GET /config/channels -> channels
      @state activeChannel: string = 'recommend'
    }
    @list 作品瀑布流 {
      @api GET /works (query: page, pageSize, channel: activeChannel) -> works
      @loading { @skeleton 卡片骨架 (count: 8) }
      @empty {
        @empty 空状态 {
          @text 暂无内容
          @button 立即投稿 { @navigate click -> /editor }
        }
      }
      @card 作品卡片 (v-for: item in works) {
        @prop item: Work
        @media 封面图 (bind: item.cover_url)
        @text 标题 (bind: item.title)
        @badge 模板 { @render when: item.is_template }
        @action 操作栏 {
          @button 点赞 (bind: item.likes_count) {
            @api POST /works/:id/like
            @mutation set works.find(w => w.id === id).likes_count += 1
            @login
          }
        }
        @navigate click -> /work/:id
      }
      @button 加载更多 {
        @api GET /works (page: nextPage) -> works(append)
        @render when: hasMore
      }
    }
  }
}
```

### 示例 3：作品详情页

```kx
@page /work/:id (作品详情) extends PosecraftLayout {
  @param id: string { from: route }

  @slot main {
    @detail 作品详情 {
      @media 图片浏览区 {
        @api GET /works/:id (bind: id) -> work
        @media 主图 (bind: work.image_url)
        @media 视频 (bind: work.video_url) { @render when: work.video_url }
      }
      @author 作者信息卡 {
        @avatar 作者头像 (bind: work.author.avatar) {
          @hover -> @popover 作者悬浮卡
          @navigate click -> /user/:id
        }
        @text 作者名 (bind: work.author.username)
        @button 关注 {
          @api POST /follow/:userId (bind: work.author.id)
          @render when: !isOwner
          @login
        }
      }
      @list 评论列表 {
        @input 评论输入框
        @button 发送 {
          @api POST /interaction/comment
          @login
        }
        @card 评论项 (v-for: comment in comments) {
          @avatar 头像 (bind: comment.author)
          @text 内容 (bind: comment.text)
        }
        @empty { @text 💬 还没有人评论 }
      }
    }
  }
}

@popover 作者悬浮卡 {
  @prop author: User
  @avatar 头像 (bind: author.avatar)
  @text 用户名 (bind: author.name)
  @stat 粉丝 (bind: author.stats.followers)
  @button 关注 { @api POST /follow/:userId @login }
}
```

---

## 三、AI 生成映射表

| KX 语法 | Vue 3 输出 |
|:---|:---|
| `@page /path (标题) extends Layout` | `router.ts` 路由 + `<router-view>` + 布局组件 |
| `@layout Name { @slot main }` | Vue SFC 布局组件 + `<slot name="main">` |
| `@header` | `<header>` sticky 顶部 |
| `@sidebar` | `<aside>` 固定宽度侧边栏 |
| `@avatar (bind: url)` | `<img>` + 圆形样式 |
| `@media (bind: url)` | `<img>` / `<video>` |
| `@card` | 卡片容器（hover 阴影） |
| `@list` | 列表容器 + 瀑布流 column |
| `@tab` | Tabs 组件 + 内部切换 |
| `@button` | `<button>` 或 `<a-button>` |
| @input | `<input>` / `<textarea>` / `<slider>` |
| `@menu` | 菜单列表 + 子菜单 |
| `@action` | 操作栏容器（flex） |
| `@modal` | Modal 弹窗（teleport body） |
| `@popover` | Popover / Tooltip（绝对定位） |
| `@badge` | 角标组件（absolute 右上角） |
| `@stat` | 统计数字 + 单位 |
| `@banner` | 轮播图（Carousel） |
| `@logo` | Logo 链接 |
| `@author` | 作者信息卡（头像+统计+关注） |
| `@toast` | Toast 轻提示 |
| `@empty` | 空状态占位组件 |
| `@skeleton` | Skeleton 骨架屏 |
| `@detail` | 详情容器（双栏布局） |
| `@social` | 社交信息流容器 |
| `@form` | `<form>` 表单容器 |
| `@state count: number = 0` | `const count = ref(0)` |
| `@prop item: Work` | `defineProps<{ item: Work }>()` |
| `@param id: string { from: route }` | `useRoute().params.id` |
| `@api GET /works -> works` | `axios.get()` + `const works = ref()` |
| `@mutation set works.find(...)` | `works.value.find(...)` 赋值 |
| `@sync filtered = works.filter(...)` | `const filtered = computed(...)` |
| `@render when: cond` | `v-if="cond"` |
| `@render unless: cond` | `v-if="!cond"` |
| `@hover -> @popover Card` | `@mouseenter` + popover 显示 |
| `@delay show: 300ms` | `setTimeout(show, 300)` |
| `@anchor @card` | 浮窗锚定目标 ref |
| `@navigate click -> /path` | `router.push('/path')` |
| `@navigate success -> /` | 成功后 `router.push('/')` |
| `@permission code` | 权限判断 `v-if` |
| `@login` | 未登录 `router.push('/login')` |
| `@loading { @skeleton }` | 加载状态分支 |
| `@empty { @empty }` | 空状态分支 |
| `@error { @toast }` | 错误状态分支 |
| `v-for: item in works` | `v-for="item in works"` |
| `bind: item.title` | `:src="item.title"` / 文本插值 |
| `-> works(append)` | push 追加到数组 |
| `-> stateName` | 返回值赋值给状态 |

---

## 四、文件组织

```
project/
├── app.kx                  # 全局布局 + 全局浮窗
├── pages/
│   ├── home.kx             # 首页频道
│   ├── work-detail.kx      # 详情 + 浮窗
│   ├── editor.kx           # 编辑器
│   ├── camera.kx           # 相机页
│   ├── login.kx            # 登录页
│   ├── mine.kx             # 我的空间
│   └── following_friends.kx # 关注/朋友
└── layouts/               # 可选：独立布局文件
    └── AppLayout.kx
```

**约定**：一个 `.kx` 文件对应一个页面或一个独立模块；`app.kx` 为全局架构入口。

---

## 五、版本历史

| 版本 | 日期 | 功能 |
|:---|:---|:---|
| v1.0 | 2025-06 | 基础指令集：`@page` `@slot` `@button` `@api` `@navigate` |
| v1.1 | 2025-08 | 新增 `@mutation` `@sync` `@render` `@state` `@prop` `@param` |
| v1.2 | 2025-10 | 新增 `@hover` `@leave` `@anchor` `@position` `@delay` `@permission` `@login` `@event` `@form` `@switch` `@summary` `@section` `@social` `@error` `@prefetch`；扩展组件类型至 23 种 |
| v2.0 (规划中) | 2026-Q1 | 服务端函数 `@server`、WebSocket `@ws`、图形绑定 `@graph`、自定义指令扩展 |
| v2.1 (规划中) | 2026-Q2 | 多端适配 `@platform`、AI 辅助生成器 `@ai`、设计稿互转 `@pen` |

---

## 六、许可证

MIT License

Copyright (c) 2025 KX Language Contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this specification and associated documentation files, to deal in the specification without restriction, including without limitation the rights to use, copy, modify, merge, publish, and distribute.
