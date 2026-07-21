# KX Documentation

KX is a **declarative page definition language** designed for AI-assisted frontend generation. Use `.kx` files to describe page structure, data flow, interaction behavior, permission rules, and business constraints — and let AI generate Vue 3 code from it.

This repository contains the KX specification, examples, and guidance for using KX as the single source of truth for page delivery.

---

## Why KX?

- **Business-first**: describe pages using terms that match product and UX intent, not raw code.
- **Model-friendly**: block-based `@<type> <name> {}` syntax that AI can parse reliably.
- **Explicit state & data**: behavior is clear through `@api`, `@state`, `@prop`, and `@mutation`.
- **Interaction-first**: conditional render, hover, popover, and navigation expressed semantically.
- **Single source of truth**: page logic, UI structure, and business rules live in one `.kx` file.

---

## Quick Start

### Step 1: Read the Spec

Full specification in [SPEC.md](./SPEC.md).

### Step 2: Create Project Structure

```
project/
├── guide/                      ← Design documents root
│   ├── index.kx                ← Entry: @ref all files
│   ├── layouts/
│   │   └── main.kx             ← Global layout
│   ├── pages/
│   │   ├── home.kx             ← Home page
│   │   ├── mine.kx             ← Profile page
│   │   └── detail.kx           ← Detail page
│   ├── models/
│   │   ├── user.kx             ← User model
│   │   ├── work.kx             ← Work model
│   │   └── api.kx              ← API definitions
│   └── components/
│       └── shared.kx           ← Shared components
├── src/                        ← Code (AI generated)
└── guide/style.md              ← Style guide
```

### Step 3: Link Files with @ref

```kx
// index.kx — project entry
@ref ./layouts/main.kx
@ref ./pages/home.kx
@ref ./pages/mine.kx
@ref ./pages/detail.kx
@ref ./models/user.kx
@ref ./models/work.kx
@ref ./models/api.kx
@ref ./components/shared.kx
```

### Step 4: Write .kx Files

Define data models in `models/`:

```kx
// models/user.kx
@model User {
  @field id: number
  @field username: string
  @field avatar: string
}

// models/api.kx
@model API {
  @api GET /api/v1/works -> works
  @api POST /api/v1/works/:id/like -> void
}
```

### Step 5: Feed to AI

A minimal working `.kx` file:

```kx
@page /discover (Discover) extends AppLayout {
  @icon SearchOutlined
  @meta title="Discover" description="Explore recommended works"

  @slot main {
    @header Search Bar {
      @input Search Input (bind: searchQuery)
      @action Filters {
        @button Recommend { @state tab: string = 'recommend' }
        @button Latest { @state tab = 'latest' }
      }
    }

    @list Works {
      @api GET /api/v1/works (query: page, keyword: searchQuery, tab: tab) -> works
      @render when: works.length > 0

      @card Work Card (v-for: item in works) {
        @prop item: Work
        @media Cover (bind: item.cover_url)
        @text Title (bind: item.title)
        @avatar Author (bind: item.author.avatar)

        @button Like {
          @api POST /api/v1/works/:id/like (body: { workId: item.id })
          @mutation set works.find(w => w.id === item.id).liked = true
          @login
        }

        @hover -> @popover Preview
      }

      @empty No results {
        @text No works found
        @button Back home { @navigate click -> / }
      }

      @loading { @skeleton Cards (count: 6) }
    }
  }
}
```

The AI generates the following Vue 3 artifacts from the spec above:

| KX Input | AI Output |
|:---|:---|
| `@page /discover (Discover)` | `router/index.ts` route + `HomeView.vue` |
| `@card Work Card { @prop item: Work … }` | `components/PoseCard.vue` sub-component |
| `@api GET /api/v1/works … -> works` | `api/works.ts` API wrapper |
| `@button Like { @mutation … @login }` | like handler + auth guard |

---

## Core Concepts

### 1. `@page` — route & metadata

Defines the route path, title, icon, SEO meta, and inherited layout.

```kx
@page /work/:id (Work Detail) extends PosecraftLayout {
  @icon EyeOutlined
  @meta title="Work Detail" description="View and interact"
  @param id: string { from: route }
  @slot main { ... }
}
```

### 2. `@layout` / `@slot` — layout & slots

Write one layout, reuse it anywhere. Child pages `extends` it and fill each `@slot`.

```kx
@layout AppLayout {
  @slot left (role: aside) { @sidebar Sidebar { … } }
  @slot top (role: header) { @header TopBar { … } }
  @slot main (role: main) { }
}
```

### 3. `@api` — request & binding

Covers HTTP methods, path params, query params, request body, variable binding, and append (`-> state(append)`).

```kx
@api GET /works (query: page, keyword) -> works
@api POST /works/:id/like (bind: id)
```

### 4. `@state` / `@prop` / `@param` — state & properties

`@state` declares reactive variables; `@prop` declares child component props; `@param` declares route parameter sources (`from: route` or `from: query`).

### 5. `@mutation` — local state updates

Supports `set` / `filter` / `remove` — assignment, filtering, and removal.

```kx
@mutation set works.find(w => w.id === id).liked = true
@mutation filter works = works.filter(w => w.id !== item.id)
```

### 6. `@sync` — computed state

Declares derived state that updates automatically when dependencies change.

```kx
@sync isVip = user.level >= 5
```

### 7. `@render` — conditional rendering

`when:` renders when true; `unless:` renders when false. Supports comparison and logical operators.

### 8. `@navigate` / `@event` — navigation & events

Covers route navigation (`@navigate click -> /path`), success callbacks (`@navigate success -> /`), and DOM event bindings.

### 9. `@login` / `@permission` — auth & guards

Three modes: login-only, permission-only, or combined. Login is checked first, then permission.

```kx
@button Create { @permission posecraft:work:create @login }
```

### 10. `@note` — business constraints

`@note` is not a comment — it is a **business constraint declaration** that generators must treat as implementation guidance.

```kx
@note Optimistic update: update UI first, rollback on API error
```

---

## Syntax Overview

### Structural Directives

| Directive | Description |
|:---|:---|
| `@page` | Page route and metadata |
| `@layout` | Reusable layout container |
| `@slot` | Named slot with `role` area type |
| `@icon` | Page / menu icon |
| `@meta` | SEO meta information |
| `@param` | Route parameter (route / query) |

### Component Directives (23 types)

| Category | Directives |
|:---|:---|
| Layout | `@header` `@sidebar` `@banner` `@detail` `@social` `@logo` |
| Content | `@card` `@list` `@text` `@media` `@badge` `@avatar` `@author` `@stat` |
| Interaction | `@button` `@input` `@tab` `@menu` `@action` `@form` |
| Feedback | `@modal` `@popover` `@toast` `@empty` `@skeleton` |

### Data & Logic Directives

| Directive | Description |
|:---|:---|
| `@api` | API request and response binding |
| `@state` | Reactive state |
| `@prop` | Component property |
| `@mutation` | Local state update (set / filter / remove) |
| `@sync` | Computed property |
| `@render` | Conditional rendering (when / unless) |
| `@navigate` | Route navigation |
| `@event` | DOM event |
| `@login` / `@permission` | Auth and permission guards |
| `@note` | Business constraint |

### Hover Interaction Directives

| Directive | Description |
|:---|:---|
| `@hover` | Hover trigger container |
| `@leave` | Leave trigger container |
| `@popover` | Popover card (absolute positioned) |
| `@delay` | Delay show / hide (`300ms`) |
| `@anchor` | Anchor popover to target component |
| `@position` | Popover position (top / bottom / left / right) |

---

## Full Example

```kx
@page /work/:id (Work Detail) extends AppLayout {
  @param id: string { from: route }

  @slot main {
    @detail Work Detail {
      @api GET /works/:id (bind: id) -> work

      @media Cover (bind: work.image_url)
      @author Author Card {
        @avatar Avatar (bind: work.author.avatar) {
          @hover -> @popover Author Popover
          @navigate click -> /user/:id
        }
        @button Follow {
          @api POST /follow/:userId (bind: work.author.id)
          @render when: !isOwner
          @login
        }
      }
      @text Title (bind: work.title)
      @action Interaction Bar {
        @button Like (bind: work.likes_count) {
          @api POST /works/:id/like
          @mutation set work.likes_count += 1
          @login
        }
        @button Collect
        @button Share
      }
      @list Comments {
        @input Comment Input
        @button Send { @api POST /interaction/comment @login }
        @card Comment Item (v-for: comment in comments) {
          @avatar Avatar (bind: comment.author)
          @text Content (bind: comment.text)
        }
      }
    }
  }
}

@popover Author Popover {
  @prop author: User
  @avatar Avatar (bind: author.avatar)
  @text Username (bind: author.name)
  @button Follow { @api POST /follow/:userId @login }
}
```

More practical examples in [example.kx](example.kx).

---

## Best Practices

1. **Single responsibility**: one `.kx` file per page or reusable module.
2. **Cross-reference**: use `@ref` in `index.kx` to link all files — AI resolves them automatically.
3. **Explicit business rules**: use `@note` to make constraints visible and actionable for AI.
4. **No hidden logic**: do not bury behavior in comments; write interactions and state clearly.
5. **Name with intent**: name blocks by purpose, not implementation detail.
6. **Visible API contracts**: declare `@api` and `@mutation` near the component that triggers them.
7. **Colocate state**: define state and its mutations in the same scope or nearest parent.

---

## Related Repos

- [kx-lang-extension](https://github.com/yijiu2025/kx-lang-extension) — VS Code syntax highlighting for `.kx` files

---

## Contribute

Contributions are welcome. You can help by:

- improving the KX specification
- adding practical `.kx` examples
- enhancing VS Code grammar support
- refining documentation and writing conventions

---

## License

[MIT](LICENSE) · Copyright (c) 2025 KX Language Contributors
