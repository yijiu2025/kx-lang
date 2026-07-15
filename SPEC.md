# KX 页面描述语言规范 v1.2

> 让 AI 读懂页面架构、自动生成代码的声明式 DSL。

## 设计原则

1. **人类可读**：像写大纲一样描述页面。
2. **AI 可解析**：结构化语法，无歧义。
3. **代码可生成**：一句描述即可生成组件、接口、路由。
4. **渐进增强**：从骨架到细节逐层展开。
5. **块级语义（v1.2）**：使用 `@<类型>` 直接表示页面块。

---

## 一、语法基础

### 1. 注释

- 单行注释：`// 注释内容`
- 多行注释：`/* 多行注释 */`

### 2. 业务说明

- `@note`：显式业务规则、约束、埋点说明。
- `@note` 内容会被 AI 识别为“必须参考”的业务信息。

示例：

```kx
@button 投稿 {
  @note 仅登录用户可见
  @note 点击需上报 event:create_work
}
```

### 3. 页面定义

```kx
@page <path> (<标题>) {
  @icon <图标名>
  @meta title="..." description="..."
  @slot main { ... }
}
```

支持：`extends` 继承布局。

```kx
@page /editor (编辑器) extends PosecraftLayout {
  @slot main { ... }
}
```

### 4. 布局与插槽

```kx
@layout <布局名> {
  @slot left { ... }
  @slot main { ... }
  @slot top { ... }
}
```

### 5. 数据绑定与参数

```kx
@prop item: Work
@state page: number = 1
@param id: string { from: route }
```

---

## 二、语义指令

### 1. 组件块指令

| 指令 | 作用 |
|:---|:---|
| `@button` | 按钮 |
| `@list` | 列表/瀑布流 |
| `@card` | 卡片 |
| `@tab` | 选项卡 |
| `@header` | 顶部栏 |
| `@sidebar` | 侧边栏 |
| `@modal` | 弹窗 |
| `@popover` | 悬浮浮窗 |
| `@badge` | 角标 |
| `@avatar` | 头像 |
| `@stat` | 统计数字 |
| `@media` | 图片/视频 |
| `@input` | 输入框 |
| `@menu` | 菜单项 |
| `@action` | 操作区 |
| `@empty` | 空状态 |
| `@skeleton` | 骨架屏 |
| `@toast` | 轻提示 |
| `@text` | 文本内容 |
| `@detail` | 详情容器 |

### 2. 逻辑与交互指令

| 指令 | 作用 |
|:---|:---|
| `@api` | 发送接口请求 |
| `@mutation` | 本地状态变更 |
| `@sync` | 同步计算 |
| `@navigate` | 路由跳转 |
| `@render` | 条件渲染 |
| `@permission` | 权限控制 |
| `@login` | 登录守卫 |
| `@hover` | 鼠标悬浮 |
| `@leave` | 鼠标离开 |
| `@anchor` | 浮窗锚点 |
| `@position` | 浮窗位置 |

### 3. `@api` 规则

```kx
@api GET /path (query: page, keyword) -> works
@api POST /item/:id/like (body: { itemId: id })
```

- 支持 HTTP 方法：`GET`、`POST`、`PUT`、`DELETE`、`PATCH`、`HEAD`、`OPTIONS`
- 支持路径变量：`/works/:id`
- 支持 `query` 参数声明
- 支持 `body` 对象声明
- `-> stateName` 表示结果写入本地状态

### 4. `@render` 规则

```kx
@render when: loading
@render when: works.length > 0
@render unless: !works.length
```

- `when:` 表示渲染条件
- `unless:` 表示反向条件
- 允许表达式、比较、逻辑运算符

### 5. `@permission` 与 `@login`

- `@permission code`：权限不足时组件隐藏
- `@login`：未登录则拦截跳转登录或显示登录提示
- 两者可以组合使用

### 6. 悬浮交互规则

#### 自动托管模式

```kx
@hover -> @popover 用户卡
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
}
```

### 7. 书写建议

- 结构清晰：`@<类型> <名称> {}` 层次分明
- 语义明确：`@note` 描述业务规则，`@render` 描述条件
- 数据显式：`@prop`、`@state`、`@param`、`@api`
- 拒绝隐式：不要把作用逻辑埋在注释里

---

## 三、扩展语法对齐

`kx-lang-extension/syntaxes/posecraft-kx.tmLanguage.json` 当前支持以下关键词：

- `@page`, `@layout`, `@slot`
- `@api`, `@navigate`, `@render`, `@permission`, `@login`
- `@state`, `@prop`, `@param`, `@mutation`, `@event`
- `@hover`, `@popover`, `@anchor`, `@position`
- `@note`, `@toast`, `@empty`, `@skeleton`
- 还有 `@list`, `@card`, `@button`, `@avatar`, `@menu`, `@action`, `@media`, `@badge`, `@text`

同时支持：

- 字符串、路径、数字、箭头 `->`
- 比较运算符 `===`, `==`, `!==`, `!=`, `>=`, `<=`, `>`, `<`
- 逻辑运算符 `||`, `&&`
- 算术运算符 `+`, `-`, `*`, `/`

---

## 四、写作规范总结

1. **保持单一页面/模块**：一个 `.kx` 文件对应一个页面或一个独立模块。
2. **结构化表达**：优先使用块级语义指令。
3. **明确数据与状态**：用 `@prop`、`@state`、`@param`、`@api`。
4. **业务说明要写 `@note`**：让 AI 理解业务边界。
5. **交互写清楚**：`@navigate`、`@render`、`@hover`、`@leave`。
6. **与扩展对齐**：语法应与 `kx-lang-extension` 语法高亮保持一致。

---

## 五、示例写法

完整示例请参考 `example.kx`。
