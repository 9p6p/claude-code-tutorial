# `cost-tracker.ts` — 成本追踪器

## 文件概述

`cost-tracker.ts` 负责追踪每个 Claude Code 会话的 **API 使用成本**。它记录 token 消耗、API 调用时间、代码行变更量，并在会话结束时输出成本摘要和持久化存储。

## 关键代码解读

### 1. 状态管理委托

```typescript
import {
  addToTotalCostState,
  getCostCounter, getTokenCounter,
  getTotalCostUSD, getTotalInputTokens,
  getTotalOutputTokens, getTotalAPIDuration,
  getModelUsage, resetCostState,
} from './bootstrap/state.js'
```

成本状态实际存储在 `bootstrap/state.ts` 中的全局单例里。`cost-tracker.ts` 是这些状态的**消费者和格式化器**。

### 2. 存储的成本状态结构

```typescript
type StoredCostState = {
  totalCostUSD: number
  totalAPIDuration: number
  totalAPIDurationWithoutRetries: number
  totalToolDuration: number
  totalLinesAdded: number
  totalLinesRemoved: number
  lastDuration: number | undefined
  modelUsage: { [modelName: string]: ModelUsage } | undefined
}
```

这个结构定义了需要持久化的成本数据——可以在恢复会话时还原之前的成本统计。

### 3. 成本计算

```typescript
import { calculateUSDCost } from './utils/modelCost.js'
```

实际的 USD 成本计算委托给 `modelCost.ts`，它维护了各个模型的价格表。

### 4. 格式化输出

成本追踪器负责在会话结束时格式化输出如下信息：
- 总 token 消耗（输入/输出/缓存读取/缓存创建）
- 按模型分组的使用量
- API 调用总时间
- 代码变更行数
- USD 总成本

## 核心导出

| 导出 | 说明 |
|------|------|
| `getTotalCost()` | 获取总 USD 成本 |
| `getTotalDuration()` | 获取总耗时 |
| `getTotalInputTokens()` | 获取总输入 token 数 |
| `getTotalOutputTokens()` | 获取总输出 token 数 |
| `formatCost()` | 格式化成本为可读字符串 |
| `resetCostState()` | 重置成本统计 |
| `getModelUsage()` | 按模型获取使用量 |

## 与其他模块的关系

```
cost-tracker.ts
  ├── bootstrap/state.ts    → 状态存储
  ├── utils/modelCost.ts    → 价格计算
  ├── utils/format.ts       → 数字格式化
  ├── costHook.ts           → React Hook 包装
  └── services/analytics/   → 遥测上报
```

## 总结

`cost-tracker.ts` 是成本透明的关键组件，让用户清楚每次会话花了多少钱。它的设计将**状态存储**（bootstrap/state）和**格式化展示**分离，职责清晰。
