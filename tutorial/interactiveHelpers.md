# `interactiveHelpers.tsx` — 交互式辅助工具

## 文件概述

`interactiveHelpers.tsx` 提供了交互式 REPL 启动过程中需要的**辅助函数和对话框基础设施**。它处理首次使用引导、信任确认、错误显示等"启动前"的交互流程。

## 关键代码解读

### 1. showDialog — 通用对话框显示

```typescript
export function showDialog<T = void>(
  root: Root,
  renderer: (done: (result: T) => void) => React.ReactNode
): Promise<T> {
  return new Promise<T>(resolve => {
    const done = (result: T): void => void resolve(result)
    root.render(renderer(done))
  })
}
```

将 React 组件的回调模式优雅地转换为 Promise：传入渲染函数，获取一个 `done` 回调，组件调用 `done` 时 Promise resolve。

### 2. exitWithError — 致命错误退出

```typescript
export async function exitWithError(
  root: Root, 
  message: string, 
  beforeExit?: () => Promise<void>
): Promise<never> {
  return exitWithMessage(root, message, {
    color: 'error',
    beforeExit,
  })
}
```

通过 Ink 渲染错误信息后退出。因为 Ink 会接管 stdout，直接 `console.error` 会被吞掉。

### 3. completeOnboarding — 完成引导

```typescript
export function completeOnboarding(): void {
  saveGlobalConfig(current => ({
    ...current,
    hasCompletedOnboarding: true,
    lastOnboardingVersion: MACRO.VERSION,
  }))
}
```

标记用户已完成首次使用引导。

### 4. showSetupScreens — 引导流程

处理首次启动时的：
- 信任对话确认
- API Key 配置
- 权限模式选择
- 频道选择

## 核心函数

| 函数 | 说明 |
|------|------|
| `showDialog()` | 通用对话框 Promise 包装 |
| `showSetupDialog()` | 带键位绑定的对话框 |
| `renderAndRun()` | 渲染并运行应用 |
| `exitWithError()` | 渲染错误并退出 |
| `exitWithMessage()` | 渲染消息并退出 |
| `showSetupScreens()` | 首次使用引导 |
| `completeOnboarding()` | 标记引导完成 |
| `getRenderContext()` | 获取渲染选项 |

## 与其他模块的关系

```
interactiveHelpers.tsx
  ├── dialogLaunchers.tsx  → 具体对话框调用此文件的基础设施
  ├── ink.ts               → TUI 渲染
  ├── bootstrap/state.ts   → 全局状态
  ├── utils/config.ts      → 配置读写
  └── main.tsx             → 被主入口调用
```

## 总结

`interactiveHelpers.tsx` 提供了"启动前"的交互基础设施。核心的 `showDialog` 模式将 React 回调转 Promise，是贯穿整个启动流程的基础范式。
