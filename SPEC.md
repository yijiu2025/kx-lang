# KX 页面描述语言规范 v1.2

> 让 AI 读懂页面架构、自动生成代码的声明式 DSL

## 设计原则

1. **人类可读**：像写大纲一样描述页面
2. **AI 可解析**：结构化语法，无歧义
3. **代码可生成**：一句描述 → 一个组件/一段逻辑
4. **渐进增强**：从骨架到细节逐层展开
5. **块级语义（v1.2）**：用 `@<类型>` 直接替代 `@component`，逻辑内聚在 `{}` 内

---

## 语法结构

### 1. 注释与说明（v1.1 引入，v1.2 强化）

业务逻辑和背景直接用注释写在架构里，AI 生成代码时会保留这些注释。

```kx
// 单行注释：描述这个组件的业务用途
@button 投稿 {   // 尾部注释
  @note 需要登录且有创作权限
}

/*
多行注释：
这里展示的是推荐流，
依据用户画像进行排序。
*/
@list Waterfall { ... }
```

**`@note` 指令（v1.2 新增）**

显式声明业务规则，AI 将其视为**强制约束**，会翻译为代码注释、条件判断或埋点。

```
@note 该接口响应较慢，需展示 Loading
@note 仅在 VIP 用户可见
@note 点击后上报埋点 click_submit
```

---

### 2. 页面定义

```
@page <路由路径> (<显示标题>) {
  @slot <区域名> (<布局类型>) {
    ...
  }
}
```

**页面元数据（v1.1 新增）**：

```
@icon <图标名>
@meta title="页面标题" (keywords="关键词") (description="描述")
```

---

### 3. 布局定义与继承（v1.1 新增）

```
@layout <布局名> {
  @slot <固定区域> { ... }
  @slot main { # 留给子页面填充 # }
}

@page <路径> extends <布局名> {
  @slot main { ... }   # 只需实现主体内容
}
```

---

### 4. 区域/插槽

```
@slot <名称> (layout: flex|grid|absolute, role: main|aside|header) {
  @component ...
}
```

---

### 5. 路由参数（v1.1 新增）

```
@param <变量名>: <类型> { from: route | query | state }
```

---

### 6. 组件（块级语义化 v1.2）

**核心变化**：不再使用 `@component` 前缀，直接用组件类型作为指令名。

#### 基本结构

```
@<组件类型> <名称> (<属性绑定>) {
  @prop <名>: <类型> = <默认值>
  @state <名>: <类型> = <默认值>
  @note <任意业务描述>
  @navigate <触发> -> <目标路由>
  @permission <权限码>
  @login                                # 登录守卫
  @api <方法> <路径> -> <本地状态>
  @render <条件>
  @event <名>(<参数>) -> <处理>
}
```

#### 组件类型关键词（直接作为指令）

| 关键词 | 含义 | 示例 |
| :--- | :--- | :--- |
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

---

### 7. 导航与事件

```
@navigate <触发> -> <目标路由>
@click -> /page/:id
@hover -> @popover 浮窗名              # 自动托管模式（v1.2）
@hover { @state show = true }          # 手动控制模式（v1.2）
@leave { @state show = false }         # 配合 hover 使用
```

#### 触发目标

| 触发写法 | 含义 |
| :--- | :--- |
| `-> /path` | 路由跳转 |
| `-> @action 名称` | 执行本地动作 |
| `-> @popover 名称` | 显示悬浮浮窗（自动绑定 hover/leave） |
| `-> @modal 名称` | 显示模态框 |
| `-> @prefetch /path` | 预加载页面数据 |
| `{ @state xxx = true }` | 修改本地状态 |

---

### 8. 权限与登录

```
@permission <权限码>                   # 细粒度权限，无权限时组件隐藏
@login                                 # 登录守卫：未登录跳转登录页，登录后继续
```

#### `@login` 三种模式

| 模式 | 写法 | 效果 |
| :--- | :--- | :--- |
| 隐藏组件 | `@permission code` | 无权限则 `v-if="false"` |
| **导航守卫** | `@login` + `@click` | 拦截 → 跳登录 → 登录后继续跳转 |
| 内容保护 | `@login` 单独使用 | 未登录显示登录引导卡，已登录显示真实内容 |

---

### 9. 数据流

```
@api GET /posecraft/v1/works -> works    # 获取数据写入本地状态
@api DELETE /posecraft/v1/works/:id      # 触发请求
```

**基础突变**

```
@mutation works DELETE id                # 本地数组删除项
```

**高级突变（v1.1）— 支持乐观更新**

```
@mutation set works.find(w => w.id === id).liked = true
@mutation set works.find(w => w.id === id).likes_count += 1
@mutation filter myWorks = myWorks.filter(w => w.id !== id)
```

**数据同步**

```
@sync tabCount = works.length            # Tab 数字同步数组长度
```

---

### 10. 条件渲染与状态容器

**基础条件**

```
@render when: loading
@render when: works.length > 0
@render when: loggedIn
@render unless: searchQuery
```

**状态容器（v1.1）— 统一处理加载/空/错误状态**

```
@loading {
  @skeleton 卡片骨架 (count: 8)
}

@empty {
  @empty 空状态 { @button 去投稿 @navigate click -> /editor }
}

@error {
  @toast "加载失败，点击重试" @event click -> @api retry
}
```

---

### 11. 悬浮交互（v1.2 双模式）

#### 模式一：自动托管（声明式）

直接映射到 `@popover`，AI 自动生成 `mouseenter` / `mouseleave` 配对逻辑。

```
@hover -> @popover <浮窗名称>
```

- **适用场景**：静态内容、无异步请求、无需延迟或精确定位。
- **AI 生成结果**：`<el-popover trigger="hover">` 或 `@mouseenter="show=true" @mouseleave="show=false"`

#### 模式二：手动控制（命令式）

通过 `@state` 显式管理显隐，配合 `@delay`、`@anchor`、`@api` 实现高级交互。

```
@hover {
  @state <状态名> = true
  @delay show: <毫秒>                   // 可选
}

@leave {
  @state <状态名> = false
  @delay hide: <毫秒>                   // 可选
}

@popover <浮窗名称> {
  @render when: <状态名>
  @anchor <锚点组件>
  @position <方位>
  // ... 可包含 @api 按需加载
}
```

- **适用场景**：异步数据加载、精确位置控制、防误触延迟、复杂动画编排。

#### 定位与延迟指令

| 指令 | 作用 | 示例 |
| :--- | :--- | :--- |
| `@anchor <组件名>` | 指定浮窗锚点，AI 生成 `getBoundingClientRect()` 定位 | `@anchor @avatar 头像` |
| `@position <方位>` | 方位选项：`top-left`、`bottom-right` 等 | `@position bottom-left` |
| `@delay <事件>: <时间>` | 延迟显示/隐藏（防误触） | `@delay hide: 300ms` |

---

## 完整实战示例

### 全局布局定义

```kx
@layout AppLayout {
  @slot top (role: header) {
    @header 主导航 {
      @button 会员功能
      @button 通知
      @button 投稿 {
        @navigate click -> /editor
        @permission work:create
      }
      @avatar 头像 {
        @hover -> @popover 用户预览卡
        @click -> /mine
        @login
      }
    }
  }

  @slot left (role: aside) {
    @logo Logo
    @sidebar 侧边栏 {
      @menu 精选 @navigate click -> /featured
      @menu 推荐 @navigate click -> /recommend
      @menu 我的 @navigate click -> /mine
    }
  }

  @slot main (role: main) {
    // 由子页面填充
  }
}
```

### 首页（带详细业务注释）

```kx
@page / (首页) extends AppLayout {
  @icon HomeOutlined
  @meta title="PoseCraft 首页" description="发现最新的 3D 角色作品"

  @slot main {
    @tab 频道Tabs {
      @api GET /config/channels -> channels
      @state activeChannel: string = 'recommend'
    }

    @banner 首页Banner {
      @api GET /banner-configs/active -> banners
      @render when: activeChannel === 'recommend' && banners.length > 0
    }

    @list 作品瀑布流 {
      @api GET /works (query: page, size: 20, channel: activeChannel) -> works

      @loading {
        @skeleton 卡片骨架 (count: 8)
      }

      @empty {
        @empty 空状态 {
          @note 新用户引导：去发布第一个作品
          @button 立即投稿 @navigate click -> /editor
        }
      }

      @error {
        @toast "加载失败，点击重试" @event click -> @api retry
      }

      @card 作品卡片 (v-for: item in works) {
        @prop item: Work

        @media 封面图 (bind: item.cover_url)
        @text 标题 (bind: item.title)

        @badge 热门 { @render when: item.hot_score > 80 }
        @badge 独家 { @render when: item.is_exclusive }

        @action 底部操作栏 {
          @button 点赞 {
            @note 乐观更新：先改 UI，接口报错再回滚
            @api POST /works/:id/like
            @mutation set works.find(w => w.id === id).liked = true
            @mutation set works.find(w => w.id === id).likes_count += 1
            @login
          }
          @button 评论 { @navigate click -> /work/:id#comment }
        }
      }

      @button 加载更多 {
        @api GET /works (page: nextPage) -> works(append)
        @render when: works.length < totalCount
      }
    }
  }
}
```

### 个人空间（含高级悬浮）

```kx
@page /mine (我的空间) extends AppLayout {
  @icon UserOutlined
  @meta title="我的空间"

  @slot main {
    @header 个人资料 {
      @api GET /user/profile -> profile

      @avatar 头像 (bind: profile.avatar) {
        @hover -> @popover 头像大图预览   // 模式一：自动
      }

      @stat 关注 (bind: followingCount) @navigate click -> /following
      @stat 粉丝 (bind: followersCount) @navigate click -> /followers

      @button 编辑资料 {
        @permission self
        @navigate click -> /profile/edit
      }
    }

    @tab 二级导航 {
      @state activeTab: string = 'works'
      @menu 作品 @sync = myWorks.length
      @menu 模板 @sync = myTemplates.length
    }

    @list 作品列表 {
      @render when: activeTab === 'works'
      @api GET /works/mine -> myWorks

      @card 作品卡片 (v-for: item in myWorks) {
        // 模式二：按需加载详情
        @state showDetail = false
        @hover { @state showDetail = true @delay show: 300ms }
        @leave { @delay hide: 200ms }

        @media 封面 (bind: item.cover)

        @popover 作品详情预览 {
          @render when: showDetail
          @anchor @card 作品卡片
          @position right-top
          @api GET /works/:id/detail (bind: item.id) -> detail
          @media 大图 (bind: detail.hd_image)
          @text 描述 (bind: detail.description)
          @button 快速点赞 {
            @api POST /works/:id/like
            @mutation set myWorks.find(w => w.id === id).likes_count += 1
          }
        }

        @button 删除 {
          @modal 确认删除 { @note "确定删除【${item.title}】吗？" }
          @api DELETE /works/:id
          @mutation filter myWorks = myWorks.filter(w => w.id !== id)
        }
      }
    }
  }
}
```

### 作品详情页（含路由参数）

```kx
@page /work/:id (作品详情) extends AppLayout {
  @icon EyeOutlined
  @meta title="作品详情"

  @param id: string { from: route }

  @slot main {
    @detail 作品详情 {
      @api GET /works/:id (bind: id) -> work

      @loading { @skeleton 全屏骨架 (count: 1) }

      @media 主图 (bind: work.image_url)
      @media 视频 (bind: work.video_url) { @render when: work.video_url }

      @author 作者区 {
        @avatar 头像 (bind: work.author.avatar)
        @text 作者名 (bind: work.author.name)
        @button 关注 { @api POST /user/follow (bind: author.id) @login }
      }

      @action 互动栏 {
        @button 点赞 (bind: work.likes_count) {
          @api POST /works/:id/like @login
          @mutation set work.likes_count += 1
        }
        @button 收藏 (bind: work.collects_count) @login
        @button 分享
      }
    }
  }
}
```

---

## AI 生成映射表

| KX 语法 | Vue 3 生成结果 |
|---|---|
| `@page /path` | `router/index.ts` 路由定义 + `views/XXXView.vue` |
| `@layout AppLayout` | 布局组件 `layouts/AppLayout.vue` |
| `@page /home extends AppLayout` | 路由注册，自动包裹布局 |
| `@icon HomeOutlined` | 路由 meta 中注册图标 |
| `@meta title="首页"` | 路由 meta 中注册标题 |
| `@param id: string` | `const id = computed(() => route.params.id)` |
| `@slot main` | `<main>` 区域或 `<router-view>` |
| `@button 提交` | `<button>提交</button>` |
| `@list` | `v-for` 循环 |
| `@card X` | 引用 `components/cards/XCard.vue` |
| `@prop item: Work` | `defineProps<{ item: Work }>()` |
| `@state loading: boolean` | `const loading = ref(false)` |
| `@api GET /path -> works` | `const { data } = await worksApi.getList()` |
| `@loading { ... }` | `v-if="loading"` 包裹内容 |
| `@empty { ... }` | `v-if="!loading && !list.length"` |
| `@error { ... }` | `v-if="error"` |
| `@navigate click -> /page` | `@click="router.push('/page')"` |
| `@hover -> @popover` | `<el-popover trigger="hover">` 或 `@mouseenter/@mouseleave` |
| `@hover { @state s = true }` | `@mouseenter="s = true"` |
| `@leave { @delay hide: 300ms }` | `@mouseleave="setTimeout(() => s=false, 300)"` |
| `@render when: show` | `v-if="show"` |
| `@anchor @avatar 头像` | 使用 ref 获取 DOM 计算定位 |
| `@permission work:create` | `v-if="authStore.hasPermission('work:create')"` |
| `@login` + `@click` | 路由守卫：`if (!loggedIn) router.push('/login')` |
| `@mutation set works[idx].liked = true` | `works.value[index].liked = true` |
| `@mutation filter works = works.filter(...)` | `works.value = works.value.filter(...)` |
| `@sync = myWorks.length` | `computed(() => myWorks.value.length)` |
| `@note 业务说明` | 生成注释 `// 业务说明` 或埋点代码 |

---

## 文件组织

```
架构文件/
├── SPEC.md                    # 本规范
├── README.md                  # 项目说明
├── layouts/
│   └── app.kx                 # 全局布局
├── pages/
│   ├── home.kx                # 首页
│   ├── mine.kx                # 个人中心
│   ├── work-detail.kx         # 作品详情
│   └── ...
└── components/
    └── shared.kx              # 公共组件定义
```

---

## 版本历史

| 版本 | 日期 | 更新内容 |
|---|---|---|
| v1.0 | 2026-01 | 初始版本：基础页面定义、组件、数据流 |
| v1.1 | 2026-07 | 新增：布局继承、乐观更新、路由参数、状态容器 |
| v1.2 | 2026-07 | 新增：块级语义（`@<类型>`）、`@note` 指令、`@hover` 双模式、`@login` 三模式、`@anchor`/`@position`/`@delay` |

---

## 许可证

MIT © PoseCraft Team
