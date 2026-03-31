# `context/` — React 上下文层

## 模块概述

`context/` 提供各种 React Context Provider 和 hooks，管理 UI 层面的横切关注点（9 个文件）。

## 文件清单

| 文件 | 说明 |
|------|------|
| `stats.tsx` | 统计指标采集（Reservoir Sampling） |
| `notifications.tsx` | 通知优先级队列 |
| `modalContext.tsx` | 模态框管理 |
| `overlayContext.tsx` | 覆盖层管理 |
| `promptOverlayContext.tsx` | 提示覆盖层 |
| `fpsMetrics.tsx` | FPS 性能指标 |
| `mailbox.tsx` | 邮箱通信 |
| `QueuedMessageContext.tsx` | 队列消息 |
| `voice.tsx` | 语音输入 |

## 核心亮点

### Stats — Reservoir Sampling

```typescript
const RESERVOIR_SIZE = 1024
// Algorithm R: 维护 p50/p95/p99 百分位
if (h.reservoir.length < RESERVOIR_SIZE) {
  h.reservoir.push(value)
} else {
  const j = Math.floor(Math.random() * h.count)
  if (j < RESERVOIR_SIZE) h.reservoir[j] = value
}
```

### Notifications — 优先级队列

```typescript
type Priority = 'low' | 'medium' | 'high' | 'immediate'
// immediate 抢占显示，支持 fold（合并同 key 通知）
```

## 总结

每个关注点一个 Provider + Hook，遵循 React 的 "Provider + Hook" 模式。水塘抽样和优先级队列是两个值得学习的算法应用。
