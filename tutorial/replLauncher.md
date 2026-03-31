# `replLauncher.tsx` — REPL 启动器

## 文件概述

`replLauncher.tsx` 是一个精简的启动器文件（22 行），负责将 REPL 界面组装并渲染到终端。它是 `main.tsx` 和实际 REPL 组件之间的桥梁。

## 完整代码解读

```typescript
import React from 'react';
import type { Root } from './ink.js';
import type { Props as REPLProps } from './screens/REPL.js';
import type { AppState } from './state/AppStateStore.js';
import type { FpsMetrics } from './utils/fpsTracker.js';

type AppWrapperProps = {
  getFpsMetrics: () => FpsMetrics | undefined;
  stats?: StatsStore;
  initialState: AppState;
};

export async function launchRepl(
  root: Root,
  appProps: AppWrapperProps,
  replProps: REPLProps,
  renderAndRun: (root: Root, element: React.ReactNode) => Promise<void>,
): Promise<void> {
  const { App } = await import('./components/App.js');
  const { REPL } = await import('./screens/REPL.js');
  await renderAndRun(root, 
    <App {...appProps}>
      <REPL {...replProps} />
    </App>
  );
}
```

### 设计要点

1. **动态导入**：`App` 和 `REPL` 通过 `await import()` 延迟加载，减少启动时间
2. **组件组装**：`App` 包裹 `REPL`，提供全局上下文（状态管理、FPS 指标）
3. **渲染委托**：实际渲染通过 `renderAndRun` 回调函数完成，解耦了启动和渲染逻辑

### 为什么需要一个单独的文件？

分离 `launchRepl` 是为了让 `main.tsx` 保持入口路由的职责，不直接处理 React 组件渲染。同时，动态导入确保了非 REPL 模式（如 `--print` 模式）不会加载 REPL 相关的沉重组件。

## 与其他模块的关系

```
replLauncher.tsx
  ├── components/App.tsx   → 根应用组件
  ├── screens/REPL.tsx     → REPL 界面（874KB！）
  ├── ink.ts               → TUI 渲染
  ├── state/AppStateStore  → 应用状态
  └── main.tsx             → 调用方
```

## 总结

`replLauncher.tsx` 虽小但关键——它是"让 Claude Code 活起来"的最后一步。通过动态导入和渲染委托，优雅地将启动逻辑与 UI 渲染解耦。
