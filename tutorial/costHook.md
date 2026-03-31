# `costHook.ts` — 成本摘要 React Hook

## 文件概述

`costHook.ts` 是一个精简的 React Hook（22 行代码），在 REPL 退出时输出成本摘要并持久化成本数据。

## 完整代码解读

```typescript
import { useEffect } from 'react'
import { formatTotalCost, saveCurrentSessionCosts } from './cost-tracker.js'
import { hasConsoleBillingAccess } from './utils/billing.js'
import type { FpsMetrics } from './utils/fpsTracker.js'

export function useCostSummary(
  getFpsMetrics?: () => FpsMetrics | undefined,
): void {
  useEffect(() => {
    const f = () => {
      if (hasConsoleBillingAccess()) {
        process.stdout.write('\n' + formatTotalCost() + '\n')
      }
      saveCurrentSessionCosts(getFpsMetrics?.())
    }
    process.on('exit', f)
    return () => {
      process.off('exit', f)
    }
  }, [])
}
```

### 工作原理

1. **注册退出回调**：在 `process.on('exit')` 上注册处理函数
2. **权限检查**：只有有计费权限的用户才会看到成本输出
3. **输出成本**：格式化并输出到 stdout
4. **持久化**：保存成本数据到磁盘
5. **清理**：组件卸载时移除事件监听器

### 为什么需要 FPS 指标？

`getFpsMetrics` 参数允许在保存成本数据时同时记录渲染性能指标。这将 UI 性能和 API 成本关联起来，便于分析"动画卡顿是否与 API 调用量有关"。

## 与其他模块的关系

```
costHook.ts
  ├── cost-tracker.ts      → 成本格式化和保存
  ├── utils/billing.ts     → 计费权限检查
  └── screens/REPL.tsx     → 在 REPL 组件中使用此 Hook
```

## 总结

`costHook.ts` 是典型的"胶水 Hook"——将 `cost-tracker` 的命令式 API 适配为 React 的声明式生命周期。虽然只有 22 行，但它确保了成本数据在任何退出路径下都能被保存。
