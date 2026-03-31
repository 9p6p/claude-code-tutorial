# `state/` — 应用状态管理

## 模块概述

`state/` 是 Claude Code 的**全局应用状态管理中心**（6 个文件）。定义了 `AppState` 类型——一个包含设置、权限、MCP 连接、通知队列、任务状态等数十个字段的巨型不可变状态对象。

## 文件清单

| 文件 | 说明 |
|------|------|
| `AppState.tsx` | React 上下文 Provider（AppStateProvider） |
| `AppStateStore.ts` | AppState 类型定义 + getDefaultAppState() |
| `store.ts` | 极简 Store 实现（35 行！） |
| `selectors.ts` | 状态选择器 |
| `onChangeAppState.ts` | 状态变更副作用 |
| `teammateViewHelpers.ts` | 团队视图辅助函数 |

## 核心设计：35 行 Store

```typescript
export function createStore<T>(initialState: T, onChange?: OnChange<T>): Store<T> {
  let state = initialState
  const listeners = new Set<Listener>()
  return {
    getState: () => state,
    setState: (updater) => {
      const prev = state
      const next = updater(prev)
      if (Object.is(next, prev)) return
      state = next
      onChange?.({ newState: next, oldState: prev })
      for (const listener of listeners) listener()
    },
    subscribe: (listener) => {
      listeners.add(listener)
      return () => listeners.delete(listener)
    },
  }
}
```

通过 `useSyncExternalStore` 与 React 集成。比 Redux 轻量 100 倍，满足 CLI 工具的需求。

## AppState 类型（部分）

```typescript
type AppState = DeepImmutable<{
  settings: SettingsJson
  mainLoopModel: ModelSetting
  toolPermissionContext: ToolPermissionContext
  kairosEnabled: boolean
  remoteSessionUrl: string | undefined
  remoteConnectionStatus: 'connecting' | 'connected' | ...
  replBridgeEnabled: boolean
  // ... 50+ 个字段
}> & {
  tasks: { [taskId: string]: TaskState }
  mcp: { clients: MCPServerConnection[]; tools: Tool[]; ... }
}
```

## 总结

`state/` 展示了"最小可用状态管理"的设计哲学——35 行 Store + DeepImmutable 类型 + useSyncExternalStore 集成。
