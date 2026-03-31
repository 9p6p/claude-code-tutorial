# `ink.ts` — TUI 渲染引擎入口

## 文件概述

`ink.ts` 是 Claude Code 的**终端 UI（TUI）渲染层入口**。它封装了 Ink 框架（一个基于 React 的终端 UI 库），并添加了主题支持。整个 Claude Code 的命令行界面——包括消息显示、输入框、工具进度——都是通过这个文件提供的 API 渲染的。

## 关键代码解读

### 1. 主题包装

```typescript
function withTheme(node: ReactNode): ReactNode {
  return createElement(ThemeProvider, null, node)
}
```

所有渲染调用都自动包裹 `ThemeProvider`，确保主题系统（亮/暗模式）全局可用。

### 2. 核心渲染 API

```typescript
export async function render(
  node: ReactNode,
  options?: NodeJS.WriteStream | RenderOptions,
): Promise<Instance> {
  return inkRender(withTheme(node), options)
}

export async function createRoot(options?: RenderOptions): Promise<Root> {
  const root = await inkCreateRoot(options)
  return {
    ...root,
    render: node => root.render(withTheme(node)),
  }
}
```

两种渲染方式：
- `render()`：一次性渲染
- `createRoot()`：创建持久根节点，支持多次更新（REPL 使用此模式）

### 3. 大量组件重导出

```typescript
export { default as Box } from './components/design-system/ThemedBox.js'
export { default as Text } from './components/design-system/ThemedText.js'
export { default as Button } from './ink/components/Button.js'
export { default as Link } from './ink/components/Link.js'
export { useInput } from './ink/hooks/use-input.js'
export { useSelection } from './ink/hooks/use-selection.js'
// ... 40+ 导出
```

`ink.ts` 作为统一的导出口，将 40+ 个组件和 hooks 重导出。其他文件只需 `import { Box, Text } from './ink.js'` 即可使用所有 UI 原语。

## 导出内容分类

| 类别 | 示例 |
|------|------|
| **布局组件** | Box, Spacer, Newline |
| **文本组件** | Text, Ansi, RawAnsi |
| **交互组件** | Button, Link |
| **主题** | ThemeProvider, useTheme, color |
| **Hooks** | useInput, useApp, useStdin, useSelection |
| **事件** | InputEvent, ClickEvent, EventEmitter |
| **焦点** | FocusManager |

## 与其他模块的关系

```
ink.ts（统一导出口）
  ├── ink/root.ts              → Ink 框架的渲染核心
  ├── ink/components/          → 基础 Ink 组件
  ├── ink/hooks/               → React hooks
  ├── ink/events/              → 事件系统
  ├── components/design-system/ → 主题化组件
  └── screens/REPL.tsx         → REPL 界面（主要消费者）
```

## 总结

`ink.ts` 是 TUI 层的"公共 API 表面"——一个 barrel 导出文件，将分散在多个目录下的组件和 hooks 统一暴露。核心设计：
- **自动主题包装**：所有渲染自动获得主题支持
- **统一导入**：消费者无需知道具体组件路径
- **TypeScript 类型安全**：所有导出都带完整类型
