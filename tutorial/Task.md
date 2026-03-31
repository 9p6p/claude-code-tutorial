# `Task.ts` — 后台任务类型系统

## 文件概述

`Task.ts` 定义了 Claude Code **后台任务系统**的核心类型。在 Claude Code 中，用户的某些操作（如运行 shell 命令、启动子 Agent、远程代理）会作为"任务"在后台运行，这个文件定义了这些任务的类型规范。

## 关键代码解读

### 1. 任务类型枚举

```typescript
export type TaskType =
  | 'local_bash'        // 本地 Bash 命令
  | 'local_agent'       // 本地子 Agent
  | 'remote_agent'      // 远程代理
  | 'in_process_teammate' // 进程内队友
  | 'local_workflow'    // 本地工作流
  | 'monitor_mcp'       // MCP 监控
  | 'dream'             // 后台推理
```

共 7 种任务类型，从简单的 shell 命令到复杂的多 Agent 协作。

### 2. 任务状态机

```typescript
export type TaskStatus =
  | 'pending'     // 待执行
  | 'running'     // 运行中
  | 'completed'   // 完成
  | 'failed'      // 失败
  | 'killed'      // 被终止

export function isTerminalTaskStatus(status: TaskStatus): boolean {
  return status === 'completed' || status === 'failed' || status === 'killed'
}
```

五个状态，其中 completed/failed/killed 是终态。`isTerminalTaskStatus` 用于判断任务是否已结束。

### 3. 任务状态基类

```typescript
export type TaskStateBase = {
  id: string
  type: TaskType
  status: TaskStatus
  description: string
  toolUseId?: string
  startTime: number
  endTime?: number
  totalPausedMs?: number
  outputFile: string
  outputOffset: number
  notified: boolean
}
```

每个任务实例的基础数据结构：
- `outputFile`：任务输出写入的磁盘文件路径
- `outputOffset`：已读取的输出偏移量
- `notified`：任务完成后是否已通知用户

### 4. Task 接口

```typescript
export type Task = {
  name: string
  type: TaskType
  kill(taskId: string, setAppState: SetAppState): Promise<void>
}
```

极简的任务接口——每种任务类型只需要实现 `kill` 方法。`spawn` 和 `render` 方法在各自的具体实现中处理。

### 5. 任务 ID 前缀

```typescript
const TASK_ID_PREFIXES: Record<string, string> = {
  local_bash: 'b',
  local_agent: 'a',
  remote_agent: 'r',
  // ...
}
```

每种任务类型有唯一的 ID 前缀，方便从 ID 快速判断任务类型。

## 核心类型汇总

| 类型 | 说明 |
|------|------|
| `TaskType` | 7 种任务类型的联合类型 |
| `TaskStatus` | 5 种状态的联合类型 |
| `TaskStateBase` | 任务实例的基础数据 |
| `Task` | 任务接口（name + type + kill） |
| `TaskHandle` | 任务句柄（taskId + cleanup） |
| `TaskContext` | 任务执行上下文 |
| `SetAppState` | 状态更新函数类型 |
| `LocalShellSpawnInput` | Shell 任务的启动参数 |

## 与其他模块的关系

```
Task.ts（类型定义）
  ├── tasks.ts                  → 任务注册表（类比 tools.ts）
  ├── tasks/LocalShellTask/     → 本地 Shell 任务实现
  ├── tasks/LocalAgentTask/     → 本地 Agent 任务实现
  ├── tasks/RemoteAgentTask/    → 远程代理任务实现
  ├── tasks/DreamTask/          → 后台推理任务
  ├── state/AppState.ts         → 任务状态存储
  └── utils/task/diskOutput.ts  → 任务输出的磁盘持久化
```

## 总结

`Task.ts` 定义了后台任务系统的类型蓝图。核心设计特点：
- **扁平的状态机**：5 个状态，3 个终态，简洁明了
- **磁盘输出**：每个任务的输出都写入文件，支持持久化和恢复
- **ID 前缀区分**：从任务 ID 即可识别类型
- **极简接口**：Task 只需实现 kill，保持最小依赖
