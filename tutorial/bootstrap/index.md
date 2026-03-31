# `bootstrap/` — 全局启动状态

## 模块概述

`bootstrap/` 包含唯一一个文件 `state.ts`（55KB，1,759 行），是整个 Claude Code 的**中央状态枢纽**。所有全局状态——会话配置、成本统计、模型使用量、遥测、权限标志——都存储在这里。

## 设计要点

```typescript
// DO NOT ADD MORE STATE HERE - BE JUDICIOUS WITH GLOBAL STATE

type State = {
  originalCwd: string
  projectRoot: string
  totalCostUSD: number
  totalAPIDuration: number
  modelUsage: { [modelName: string]: ModelUsage }
  sessionId: SessionId
  meter: Meter | null           // OpenTelemetry
  isRemoteMode: boolean
  invokedSkills: Map<string, SkillInfo>
  // ... 100+ 个字段
}
```

### 核心特点

1. **单例模式**：模块级 `state` 变量，通过 getter/setter 函数访问
2. **跨模块共享**：几乎所有模块都从这里读取/写入状态
3. **显式警告**：注释明确说"不要随意添加全局状态"
4. **Sticky Latch**：某些状态一旦设置就不再改变（如 `afkModeHeaderLatched`），避免破坏 prompt cache

### 状态分类

| 类别 | 示例字段 |
|------|---------|
| 会话 | sessionId, projectRoot, cwd |
| 成本 | totalCostUSD, totalAPIDuration, modelUsage |
| 遥测 | meter, loggerProvider, tracerProvider |
| 权限 | sessionBypassPermissionsMode, sessionTrustAccepted |
| 缓存 | systemPromptSectionCache, promptCache1hAllowlist |
| UI | agentColorMap, slowOperations |

## 总结

`bootstrap/state.ts` 是项目的"全局内存"。虽然 100+ 字段的全局状态不是最优雅的设计，但对于 CLI 工具来说这种方式简单高效。Sticky Latch 模式是一个值得学习的优化技巧——避免频繁切换破坏 API prompt cache。
