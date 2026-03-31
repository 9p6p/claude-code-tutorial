# `Tool.ts` — 工具类型系统

## 文件概述

`Tool.ts` 是 Claude Code 工具系统的**类型定义核心**，大约 300 行代码。它定义了所有工具必须遵循的接口契约、工具执行上下文（`ToolUseContext`）、权限系统类型和工具进度跟踪类型。可以把它理解为"工具系统的蓝图"。

## 关键代码解读

### 1. 工具输入 Schema 类型

```typescript
export type ToolInputJSONSchema = {
  [x: string]: unknown
  type: 'object'
  properties?: {
    [x: string]: unknown
  }
}
```

每个工具都需要定义一个 JSON Schema 来描述其输入参数，这个 Schema 会发送给 Claude API，让模型知道如何正确调用工具。

### 2. 权限上下文 — ToolPermissionContext

```typescript
export type ToolPermissionContext = DeepImmutable<{
  mode: PermissionMode
  additionalWorkingDirectories: Map<string, AdditionalWorkingDirectory>
  alwaysAllowRules: ToolPermissionRulesBySource
  alwaysDenyRules: ToolPermissionRulesBySource
  alwaysAskRules: ToolPermissionRulesBySource
  isBypassPermissionsModeAvailable: boolean
  isAutoModeAvailable?: boolean
  shouldAvoidPermissionPrompts?: boolean
  prePlanMode?: PermissionMode
}>
```

这是 Claude Code 权限系统的核心数据结构。注意使用了 `DeepImmutable` 确保不可变性：
- `mode`：当前权限模式（default/auto/bypass）
- `alwaysAllowRules`/`alwaysDenyRules`/`alwaysAskRules`：按来源分组的权限规则
- `shouldAvoidPermissionPrompts`：后台 Agent 用，自动拒绝所有权限请求

### 3. 工具执行上下文 — ToolUseContext

```typescript
export type ToolUseContext = {
  options: {
    commands: Command[]
    tools: Tools
    mainLoopModel: string
    mcpClients: MCPServerConnection[]
    thinkingConfig: ThinkingConfig
    maxBudgetUsd?: number
    refreshTools?: () => Tools
    // ...
  }
  abortController: AbortController
  readFileState: FileStateCache
  getAppState(): AppState
  setAppState(f: (prev: AppState) => AppState): void
  messages: Message[]
  // ... 30+ 个属性
}
```

这是**每次工具调用时传递的上下文对象**，包含了工具运行所需的一切：
- 应用状态的读写器（`getAppState` / `setAppState`）
- 中断控制器（`abortController`）
- 文件缓存（`readFileState`）
- UI 操作（`setToolJSX`、`sendOSNotification`）
- 消息历史（`messages`）

### 4. 验证结果类型

```typescript
export type ValidationResult =
  | { result: true }
  | { result: false; message: string; errorCode: number }
```

工具可以验证其输入参数，返回成功或带错误信息的失败结果。

### 5. SetToolJSX — UI 注入能力

```typescript
export type SetToolJSXFn = (
  args: {
    jsx: React.ReactNode | null
    shouldHidePromptInput: boolean
    shouldContinueAnimation?: true
    showSpinner?: boolean
  } | null,
) => void
```

工具可以通过这个函数向 REPL UI 注入自定义 JSX 内容（如配置界面、进度条等），这赋予了工具"临时接管界面"的能力。

## 核心类型汇总

| 类型 | 说明 |
|------|------|
| `Tool` | 工具接口定义（名称、描述、Schema、执行函数） |
| `Tools` | `Tool[]` 的别名 |
| `ToolUseContext` | 工具执行时的完整上下文 |
| `ToolPermissionContext` | 权限检查上下文 |
| `ToolInputJSONSchema` | 工具参数的 JSON Schema |
| `ValidationResult` | 参数验证结果 |
| `SetToolJSXFn` | UI 注入回调 |
| `CompactProgressEvent` | 消息压缩进度事件 |
| `QueryChainTracking` | 查询链追踪（用于多轮调用） |

## 与其他模块的关系

```
Tool.ts（类型定义中心）
  ├── tools/*.ts          → 具体工具实现此接口
  ├── tools.ts            → 注册使用这些类型
  ├── query.ts            → 使用 ToolUseContext 执行查询
  ├── state/AppState.ts   → 提供应用状态
  ├── types/permissions.ts → 权限类型来源
  └── types/tools.ts      → 工具进度类型来源
```

## 总结

`Tool.ts` 定义了工具系统的"合同"——任何新工具都必须满足这里定义的接口。核心设计亮点：
- **DeepImmutable 权限上下文**：通过类型系统强制不可变性
- **丰富的 ToolUseContext**：一个对象包含工具所需的所有依赖
- **UI 注入能力**：工具不只是执行逻辑，还能临时操控界面
- **类型导出策略**：从多个子模块收集类型并统一重导出，避免循环依赖
