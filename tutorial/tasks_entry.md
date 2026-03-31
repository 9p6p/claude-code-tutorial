# `tasks.ts` — 任务注册表

## 文件概述

`tasks.ts` 是后台任务系统的**注册表**，与 `tools.ts` 对工具的角色相同。它维护所有可用的任务类型实现，并提供按类型查找任务的功能。

## 关键代码解读

### 1. 任务注册

```typescript
import { DreamTask } from './tasks/DreamTask/DreamTask.js'
import { LocalAgentTask } from './tasks/LocalAgentTask/LocalAgentTask.js'
import { LocalShellTask } from './tasks/LocalShellTask/LocalShellTask.js'
import { RemoteAgentTask } from './tasks/RemoteAgentTask/RemoteAgentTask.js'

export function getAllTasks(): Task[] {
  const tasks: Task[] = [
    LocalShellTask,
    LocalAgentTask,
    RemoteAgentTask,
    DreamTask,
  ]
  if (LocalWorkflowTask) tasks.push(LocalWorkflowTask)
  if (MonitorMcpTask) tasks.push(MonitorMcpTask)
  return tasks
}
```

4 个核心任务 + 2 个条件任务（通过 Feature Flag 控制）。

### 2. 按类型查找

```typescript
export function getTaskByType(type: TaskType): Task | undefined {
  return getAllTasks().find(t => t.type === type)
}
```

简单直接的查找函数，用于根据任务类型获取对应的任务实现。

## 任务类型说明

| 任务 | 类型 | 说明 |
|------|------|------|
| `LocalShellTask` | `local_bash` | 在本地执行 shell 命令 |
| `LocalAgentTask` | `local_agent` | 本地子 Agent 任务 |
| `RemoteAgentTask` | `remote_agent` | 远程代理任务 |
| `DreamTask` | `dream` | 后台推理任务 |
| `LocalWorkflowTask` | `local_workflow` | 本地工作流（Feature Flag） |
| `MonitorMcpTask` | `monitor_mcp` | MCP 监控（Feature Flag） |

## 与其他模块的关系

```
tasks.ts（注册表）
  ├── Task.ts                   → 类型定义
  ├── tasks/LocalShellTask/     → Shell 任务实现
  ├── tasks/LocalAgentTask/     → Agent 任务实现
  ├── tasks/RemoteAgentTask/    → 远程任务实现
  ├── tasks/DreamTask/          → 后台推理实现
  └── tools/AgentTool/          → AgentTool 触发创建任务
```

## 总结

`tasks.ts` 是一个精简的注册表（40 行），遵循与 `tools.ts` 相同的模式：函数式获取 + Feature Flag 条件加载。
