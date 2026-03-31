# `coordinator/` — 多 Agent 协调器模式

## 模块概述

`coordinator/` 模块实现了 Claude Code 的**协调器模式**（Coordinator Mode）——一种多 Agent 编排模式，其中一个协调者 AI 将任务委派给多个 worker Agent 并行执行。这是 Claude Code 最强大的能力之一。

## 文件清单

| 文件 | 说明 |
|------|------|
| `coordinatorMode.ts` | 协调器模式的核心逻辑 |

## 核心功能

### 1. 模式检测

```typescript
export function isCoordinatorMode(): boolean {
  if (feature('COORDINATOR_MODE')) {
    return isEnvTruthy(process.env.CLAUDE_CODE_COORDINATOR_MODE)
  }
  return false
}
```

通过编译时 Feature Flag + 运行时环境变量双重控制。

### 2. Worker 上下文生成

```typescript
export function getCoordinatorUserContext(
  mcpClients: ReadonlyArray<{ name: string }>,
  scratchpadDir?: string,
): { [k: string]: string } {
  // 生成 worker 可用工具列表
  // 添加 MCP 服务器信息
  // 添加 Scratchpad 目录信息
}
```

告诉协调器 AI "你的 worker 可以用哪些工具"。

### 3. 协调器系统提示（核心）

```typescript
export function getCoordinatorSystemPrompt(): string {
  return `You are Claude Code, an AI assistant that orchestrates...
  
  ## 1. Your Role
  - Help the user achieve their goal
  - Direct workers to research, implement and verify
  
  ## 4. Task Workflow
  | Phase | Who | Purpose |
  |-------|-----|---------|
  | Research | Workers (parallel) | 调查代码库 |
  | Synthesis | You (coordinator) | 综合发现，制定方案 |
  | Implementation | Workers | 执行变更 |
  | Verification | Workers | 测试验证 |
  `
}
```

系统提示定义了完整的 4 阶段工作流：**研究 → 综合 → 实现 → 验证**。核心原则是"并行是你的超能力"。

### 4. 会话模式匹配

```typescript
export function matchSessionMode(
  sessionMode: 'coordinator' | 'normal' | undefined,
): string | undefined {
  // 恢复会话时，自动切换到匹配的模式
}
```

## 与其他模块的关系

```
coordinator/
  ├── tools.ts          → 工具过滤（协调器只有 Agent/TaskStop/SendMessage）
  ├── Tool.ts           → AgentTool 创建 worker
  ├── bootstrap/state   → 环境变量读取
  └── main.tsx          → 条件加载此模块
```

## 总结

协调器模式是 Claude Code 的"指挥官模式"——AI 不再亲自执行代码操作，而是指挥多个 worker 并行工作。核心设计亮点是精心编写的系统提示，定义了清晰的分阶段工作流和 worker 管理规范。
