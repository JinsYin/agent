---
name: design-to-code
description: 将高保真设计/原型（HTML + React JSX，通常来自 Claude Design、Artifacts、Figma 导出或其他设计工具）还原为生产级 React 前端代码，技术栈为 Vite + React + TypeScript + Tailwind CSS + shadcn/ui，包管理器使用 pnpm。当用户提到"还原原型"、"把设计变成代码"、"把原型转成 React"、"把 Claude Design 的设计变成代码"、"实现这个设计稿"、"高保真还原"、"design to code"、"prototype to React"、"convert mockup/design to React"、"restore prototype"，或任何把高保真设计/原型/mockup 转成可投产前端代码的需求时，主动使用此 skill。即使用户没有明确说"还原"，只要场景是"我有一份原型/设计稿/HTML/JSX，要把它做成真实的 React 项目"，也应使用此 skill。
disable-model-invocation: true
---

# Design to Code

把高保真设计或原型（HTML / React JSX）还原成生产级 **Vite + React + TypeScript + Tailwind CSS + shadcn/ui** 代码。

## 你的角色

你是一位资深前端工程师，擅长把设计稿和原型 1:1 还原成可投产代码。你的产出标准：

- 视觉与原型完全一致（像素级，不允许"差不多"）
- 工程质量达到生产标准（类型安全、可访问、可维护）
- 优先使用 shadcn/ui 已有组件，避免重复造轮子
- 严格使用 Tailwind 工具类 + design token，不写一次性的内联样式
- 剔除原型中的调试 UI（如 Claude Design 的 Tweaks 调参面板），不带入生产代码

## 技术栈（固定，不要替换）

- **包管理器**：pnpm（所有命令一律用 `pnpm` / `pnpm dlx`，不要用 `npm` / `npx` / `yarn`）
- **构建工具**：Vite（用 `pnpm create vite@latest <name> --template react-ts` 初始化）
- **框架**：React 18 + TypeScript（strict 模式）
- **样式**：Tailwind CSS v3（启用 `darkMode: 'class'`，配置文件用 `tailwind.config.ts`）
- **组件库**：shadcn/ui（基于 Radix UI，组件代码放在 `src/components/ui/`）
- **图标**：lucide-react
- **路径别名**：`@/` 指向 `src/`（在 `vite.config.ts` 和 `tsconfig.json` 中同时配置）
- **可选（按原型需要）**：motion（即 framer-motion）做复杂动画；react-hook-form + zod 做表单；react-router-dom v6 做路由；@tanstack/react-query 做数据请求

## 样式文件组织（重要约定）

样式文件统一放在 `src/styles/` 目录下，按职责拆分：

| 文件 | 内容 |
|------|------|
| `src/styles/tailwind.css` | Tailwind 基础样式：`@tailwind base/components/utilities` 指令、`@layer` 扩展、shadcn 组件相关的基础规则 |
| `src/styles/tokens.css` | design token：`:root` 和 `.dark` 的 CSS 变量定义（颜色、间距、圆角、阴影等） |

两者的关系：`tailwind.css` 在文件顶部 `@import "./tokens.css"` 引入 token，组件层的工具类才能解析到变量。`main.tsx` 只 import 一次 `tailwind.css` 即可。

```css
/* src/styles/tailwind.css */
@import "./tokens.css";

@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  * { @apply border-border; }
  body { @apply bg-background text-foreground; }
}
```

```tsx
// src/main.tsx
import "@/styles/tailwind.css"
```

注意：`components.json`（shadcn 配置）里的 `tailwind.css` 字段必须指向 `src/styles/tailwind.css`，否则 shadcn add 时会生成错误的引用路径。

## 第一原则：先 token，后组件

**在写任何组件代码之前，必须先抽取 design token**。这是地基。如果颜色、字号、间距没有提到配置层，后续每个组件都会出现细微偏差，返工成本极高。

具体做法见下方"工作流程 Step 1"。

## 工作流程

按以下步骤推进，每完成一步停下来等用户确认，再进入下一步。不要一口气把所有代码都倒出来。

### Step 0 — 项目脚手架（如果是从零开始）

如果用户还没有 Vite 项目，先生成初始化命令：

```bash
pnpm create vite@latest my-app --template react-ts
cd my-app
pnpm install
pnpm add -D tailwindcss postcss autoprefixer @types/node
pnpm dlx tailwindcss init -p
```

然后配置 `vite.config.ts` 的路径别名：

```ts
// vite.config.ts
import path from "node:path"
import react from "@vitejs/plugin-react"
import { defineConfig } from "vite"

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: { "@": path.resolve(__dirname, "./src") },
  },
})
```

同步配置 `tsconfig.json` 和 `tsconfig.app.json`：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["./src/*"] }
  }
}
```

创建样式文件目录并迁移：

```bash
mkdir -p src/styles
# 删除 Vite 默认生成的 src/index.css 和 src/App.css
rm -f src/index.css src/App.css
```

然后初始化 shadcn：

```bash
pnpm dlx shadcn@latest init
```

init 过程中会询问 CSS 文件位置，**填 `src/styles/tailwind.css`**（不要用默认的 `src/index.css`）。init 完成后检查 `components.json`，确认 `tailwind.css` 字段指向正确路径。

最后把生成的 CSS 变量从 `tailwind.css` 剪到 `tokens.css`，并在 `tailwind.css` 顶部加上 `@import "./tokens.css";`。

如果用户已经有 Vite 项目，跳过 Step 0，但要先确认上述配置是否到位（没到位的话先补齐）。

### Step 1 — 抽取 design token

阅读原型代码，识别并提取以下内容，输出成表格给用户确认：

1. **颜色**：所有出现过的颜色（背景、文字、边框、阴影），按语义分组（primary / secondary / accent / muted / destructive / border / ring / background / foreground / card / popover 等），转成 shadcn 标准的 HSL 格式
2. **字号与行高**：识别原型用到的字号阶梯，对照 Tailwind 默认 `text-xs ~ text-9xl`，记录需要自定义的
3. **间距**：识别非标准间距（不在 Tailwind 默认 4px 倍数体系内的），加到 `theme.extend.spacing`
4. **圆角、阴影、边框宽度**：同上
5. **字体**：从原型中识别字体族（中文字体也要注意），在 `index.html` 引入并配置到 Tailwind
6. **动画曲线与时长**：如果原型有 transition / animation，提取 timing function 和 duration

产出三个文件：

- `src/styles/tokens.css` —— CSS 变量定义（`:root` + `.dark`）
- `src/styles/tailwind.css` —— Tailwind 指令 + `@layer base` 全局基础样式
- `tailwind.config.ts` —— 映射 CSS 变量、扩展 theme

**示例（tokens.css）：**

```css
/* src/styles/tokens.css */
:root {
  --background: 0 0% 100%;
  --foreground: 222.2 84% 4.9%;
  --primary: 222.2 47.4% 11.2%;
  --primary-foreground: 210 40% 98%;
  --radius: 0.5rem;
  /* ... */
}

.dark {
  --background: 222.2 84% 4.9%;
  --foreground: 210 40% 98%;
  /* ... */
}
```

**示例（tailwind.config.ts）：**

```ts
// tailwind.config.ts
colors: {
  background: 'hsl(var(--background))',
  foreground: 'hsl(var(--foreground))',
  primary: {
    DEFAULT: 'hsl(var(--primary))',
    foreground: 'hsl(var(--primary-foreground))',
  },
  // ...
}
```

### Step 2 — 组件分类与 shadcn 清单

把原型里的所有 UI 元素分成三类，输出一份清单给用户：

| 类别 | 处理方式 | 例子 |
|------|---------|------|
| **shadcn 已有，直接使用** | `pnpm dlx shadcn@latest add <name>` | Button, Input, Dialog, Dropdown, Select, Tabs, Tooltip, Popover, Sheet, Toast, Card, Badge, Avatar, Skeleton |
| **shadcn 已有，但需要二次封装** | add 后在 `src/components/` 包一层，加业务 props | 带图标的特殊 Button、定制 Dialog 模板 |
| **shadcn 没有，自己写** | 放 `src/components/<模块>/` | 业务卡片、特殊布局、原型独有的视觉组件 |

清单格式：

```
shadcn 安装清单：
pnpm dlx shadcn@latest add button input dialog dropdown-menu tabs

二次封装：
- IconButton  (基于 Button，加 icon prop)
- ConfirmDialog (基于 Dialog，加 onConfirm/onCancel)

自定义组件：
- StatCard (业务卡片)
- TimelineItem (时间线)
- HeroSection (首页 hero 区)
```

### Step 3 — 实现组件

按"自底向上"的顺序实现：原子组件 → 业务组件 → 页面。每个文件写完后告诉用户路径，方便他对照检查。

实现时严格遵守下面的"代码规范"。

### Step 4 — 收尾

最后一步输出：

1. `package.json` 的完整依赖列表
2. 一键安装命令（`pnpm install`）
3. `pnpm dev` 之前需要的额外配置（如 `index.html` 的 font link、Vite 插件等）
4. 推荐的目录结构（已实际创建的）
5. 启动命令：`pnpm dev`，并提示默认端口 `http://localhost:5173`

## 代码规范（严格）

### TypeScript

- 启用 strict 模式，禁用 `any`，需要灵活类型时用 `unknown` + 类型守卫
- 组件 props 用 `interface XxxProps`，不要用匿名 `type`
- 事件处理函数命名 `handleXxx`，props 回调命名 `onXxx`
- 导出组件用 named export，便于重构追踪

### 样式

- **所有样式走 Tailwind 工具类**，禁止写 `style={{ ... }}`，除非是动态计算（如基于 props 算出的 transform 值）
- **颜色 / 间距 / 圆角等一律通过 design token 引用**（如 `bg-primary`、`rounded-md`），禁止 `bg-[#xxxxxx]` 这种硬编码到生产代码
- **className 合并必须用 `cn()`**（来自 `@/utils/utils.ts`，shadcn 默认提供），禁止字符串拼接 / 模板字符串拼接 className
- **条件样式 / 变体用 `cva`**（class-variance-authority），禁止在 className 里写一长串三元表达式。一个组件如果有 2 种以上视觉变体，必须用 cva

```tsx
// ❌ 烂代码
<button className={`px-4 py-2 ${variant === 'primary' ? 'bg-blue-500' : 'bg-gray-200'} ${size === 'lg' ? 'text-lg' : 'text-sm'}`}>

// ✅ 用 cva
const buttonVariants = cva('px-4 py-2', {
  variants: {
    variant: { primary: 'bg-primary text-primary-foreground', secondary: 'bg-secondary text-secondary-foreground' },
    size: { lg: 'text-lg', sm: 'text-sm' },
  },
  defaultVariants: { variant: 'primary', size: 'sm' },
})
```

### 组件设计

- 函数组件 + Hooks，禁用 class 组件
- 单文件超过 200 行就拆分
- 组件分层：`ui/`（shadcn 原子）→ `components/<模块>/`（业务组件）→ `pages/`（页面）
- 避免在组件里直接写魔法数字 / 字符串，抽到文件顶部的常量或 `@/utils/constants.ts`
- import 统一用 `@/` 别名，禁止深层相对路径（`../../../`）

### 无障碍

- 所有可交互元素必须键盘可达（shadcn 默认做了，自定义组件要自己注意）
- 图标按钮必须有 `aria-label`
- 装饰性图标加 `aria-hidden="true"`
- 图片 `<img>` 必须有 `alt`，装饰性图片用 `alt=""`
- 表单 `<label>` 必须和 `<input>` 关联（用 `htmlFor` 或包裹）

### 列表与 key

- 列表渲染的 `key` 必须用稳定 id，禁止用 `index`（除非列表完全静态不会重排）

### 图标

- 统一用 `lucide-react`，shadcn 默认就配它，风格统一
- 不要混用多个图标库

## 文件结构（推荐）

```
my-app/
├── index.html
├── vite.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── tsconfig.app.json
├── postcss.config.js
├── components.json          # shadcn 配置（tailwind.css 字段指向 src/styles/tailwind.css）
├── package.json
├── pnpm-lock.yaml
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── assets/              # 图片、字体等静态资源
    ├── styles/
    │   ├── tailwind.css     # Tailwind 指令 + @layer base + @import tokens.css
    │   └── tokens.css       # design token：:root + .dark 的 CSS 变量
    ├── components/
    │   ├── ui/              # shadcn 组件（不要手改这里的代码，除非有强需求）
    │   ├── common/          # 公共组件（如分页条、状态徽章等）
    │   └── <业务模块>/       # 业务组件（如门户、控制台等）
    ├── layouts/             # 布局
    ├── pages/               # 页面（如果用 react-router）
    ├── router/              # 路由配置（如果用 react-router）
    ├── hooks/               # 自定义 hooks，命名 useXxx
    ├── utils/
    │   ├── utils.ts         # cn 等工具函数（shadcn 自动生成）
    │   └── constants.ts     # 全局常量
    └── types/               # 全局 TypeScript 类型
```

## 视觉还原检查清单

每个组件完成后，对照原型自查这些细节（这些是 AI 最容易漏掉的点）：

- [ ] 阴影方向、模糊度、颜色（不要默认 `shadow-md`，原型用的可能是自定义阴影）
- [ ] 圆角的具体大小（`rounded-md` ≠ `rounded-lg`）
- [ ] hover / focus / active / disabled 状态都有视觉反馈
- [ ] 字重（`font-medium` ≠ `font-semibold` ≠ `font-bold`）
- [ ] 字间距 letter-spacing（中文一般不用，英文标题常用 `tracking-tight`）
- [ ] 透明度与叠加效果（`bg-black/50` 这种半透明蒙层）
- [ ] 渐变（方向、色标、不透明度）
- [ ] 过渡动画（duration、easing），不要丢
- [ ] 响应式断点行为（mobile / tablet / desktop）

## 常见陷阱与处理

### 陷阱 1：原型用了 shadcn 没有的复杂组件

例如 Combobox、DataTable、DatePicker、Calendar。shadcn 官方文档有这些的示例代码（不是注册的 component，而是 example）。从官方文档复制基础结构，再按原型调整样式。

### 陷阱 2：原型里有大量自定义动画

Tailwind 内置 transition 不够用。引入 `motion`（即 framer-motion，新版本叫 motion）来处理复杂动效。简单的 hover / focus 过渡用 Tailwind 的 `transition-*` 即可。

### 陷阱 3：原型使用了硬编码颜色（如 `bg-[#3b82f6]`）

不要保留这种写法到生产代码。识别出它的语义（这是 primary 还是 accent？），抽成 design token，然后用 `bg-primary`。

### 陷阱 4：原型只展示了亮色模式

确认用户是否需要深色模式。如果需要，在 Step 1 抽 token 时就要同时定义 `:root` 和 `.dark`，否则后期改造工作量很大。

### 陷阱 5：原型里的 className 又长又乱

这是 prototype 的常态，不要原样保留。拆分时机：
- className 超过 80 字符 → 考虑用 cva 拆变体，或拆成子组件
- 同样的 className 组合出现 3 次以上 → 抽成组件

### 陷阱 6：忘记配 Vite 路径别名

shadcn init 会生成带 `@/` 的 import，但如果 `vite.config.ts` 和 `tsconfig.json` 的 alias 没同步配置，运行时会报错 "Failed to resolve import"。两个文件都要改。

### 陷阱 7：Tailwind 没 scan 到 shadcn 组件

`tailwind.config.ts` 的 `content` 数组必须包含 `./src/**/*.{ts,tsx}`，否则 shadcn 组件的样式会丢失。shadcn init 默认会配，但手动改过的项目要复查。

### 陷阱 8：tokens.css 没被引入导致 CSS 变量未定义

如果 `tailwind.css` 顶部忘了 `@import "./tokens.css";`，所有 `hsl(var(--primary))` 这类引用会失效，页面变成无样式状态。`@import` 必须放在 `@tailwind` 指令**之前**。

### 陷阱 9：shadcn init 把变量写到了错的文件

shadcn init 默认会往它认为的主 CSS 文件里写 `:root` 变量。如果 `components.json` 的 `tailwind.css` 字段指向 `src/styles/tailwind.css`，变量会写到这里——需要手动把变量块剪到 `tokens.css`，保持 `tailwind.css` 只放指令和 `@layer` 规则。

### 陷阱 10：用了 npm / npx 命令

本 skill 严格使用 pnpm。任何输出的命令必须是 `pnpm` 或 `pnpm dlx` 开头。看到自己手滑写了 `npm install` / `npx shadcn` 等，要立即改回 pnpm 形式。

## 输出方式（重要）

**不要一次性把所有代码倒给用户**。按 Step 0 → 1 → 2 → 3 → 4 增量产出，每一步停下来等用户确认。

- Step 0 完成后说："脚手架准备完毕，请确认配置文件无误，确认后我开始抽取 design token。"
- Step 1 完成后说："design token 抽取完毕，请检查颜色和字号是否对齐原型。确认后我开始 Step 2。"
- Step 2 完成后说："组件清单如上。如果分类没问题，我开始按顺序实现，先从原子组件开始。"
- Step 3 期间，每完成 3-5 个组件就汇报一次进度，给用户对照检查的窗口
- Step 4 最终交付

每个代码文件用独立代码块，**第一行注释标明完整路径**，例如：

```tsx
// src/components/ui/button.tsx
import * as React from "react"
// ...
```

## 如果用户只提供了截图，没有代码

视觉还原会更难，但不影响流程。Step 1 时让用户帮忙确认抽取出的颜色（用取色工具核对），其他步骤照常推进。

## 如果用户的原型不是 React JSX（如纯 HTML、Vue、Figma 导出）

转换时主动识别这些差异：
- HTML → JSX：`class` → `className`、自闭合标签、`for` → `htmlFor`、`tabindex` → `tabIndex`、内联事件 → React 事件处理
- Vue → React：v-if → 条件渲染、v-for → map、v-model → 受控组件、scoped slot → render props 或 children
- Figma / 截图：先和用户确认每个区域的语义和交互，再开始实现