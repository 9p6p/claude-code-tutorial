# `moreright/` — 内部功能存根

## 模块概述

`moreright/` 在外部构建中是一个**存根模块**——真正的 `useMoreRight` hook 仅存在于 Anthropic 内部版本中。

## 文件：`useMoreRight.tsx`

```typescript
export function useMoreRight(_args: {
  enabled: boolean
  setMessages: (action: M[] | ((prev: M[]) => M[])) => void
  inputValue: string
  setInputValue: (s: string) => void
  setToolJSX: (args: M) => void
}): {
  onBeforeQuery: (...) => Promise<boolean>
  onTurnComplete: (...) => Promise<void>
  render: () => null
} {
  return {
    onBeforeQuery: async () => true,
    onTurnComplete: async () => {},
    render: () => null,
  }
}
```

存根返回空操作，确保外部构建编译通过。

## 总结

典型的"存根模式"（Stub Pattern）——让外部版本与内部版本保持接口兼容。
