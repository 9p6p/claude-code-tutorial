# `cli/` — CLI 输出与通信层

## 模块概述

CLI 的输出渲染和 I/O 通信层（8 个文件 + handlers/ 和 transports/ 子目录），负责将内部消息转换为终端可读格式。

## 文件清单

| 文件 | 大小 | 说明 |
|------|------|------|
| `print.ts` | **208KB** | 消息→终端渲染引擎（巨型文件！） |
| `structuredIO.ts` | 28KB | SDK 结构化 JSON I/O（`-p` 模式） |
| `update.ts` | 14KB | 自动更新检查 |
| `remoteIO.ts` | 10KB | 远程 I/O |
| `exit.ts` | 1.3KB | CLI 退出辅助（`never` 类型应用） |
| `ndjsonSafeStringify.ts` | 1.4KB | NDJSON 安全序列化 |
| `handlers/mcp.tsx` | 55KB | MCP 管理子命令（`claude mcp ...`） |
| `handlers/plugins.ts` | 30KB | 插件管理子命令 |
| `handlers/auth.ts` | 11KB | 认证子命令 |

## 核心文件：`print.ts` — 208KB 渲染引擎

项目中数一数二的大文件（**5,595 行**），是非交互式模式的核心。

### 依赖图谱

前 200 行全是 import 语句，依赖几乎所有模块：

```
print.ts 依赖全景
  ├── 命令系统: Command, Tool, Tools
  ├── 消息类型: Message, SDKMessage, SDKUserMessage
  ├── 状态管理: AppState, SessionState
  ├── I/O: StructuredIO, RemoteIO
  ├── 工具系统: tools, analytics, mcp, permissions
  ├── 会话管理: conversationRecovery, sessionStorage
  ├── 输出样式: streamlinedTransform, outputStyles, fastMode
  ├── MCP 通道: ChannelMessageNotification
  ├── Hook 系统: hookEvents, AsyncHookRegistry
  └── 其他: idleTimeout, gracefulShutdown, cleanupRegistry
```

### 核心功能区

| 功能 | 说明 |
|------|------|
| **结构化输出** | `StructuredIO` + `streamJsonStdoutGuard` — SDK JSON 模式 |
| **命令队列** | `enqueue/dequeue` + `subscribeToCommandQueue` — 消息排队 |
| **工具权限** | `ToolPermissionContext` + `PermissionPromptTool` — 权限处理 |
| **会话恢复** | `loadConversationForResume` — `--resume` 模式 |
| **空闲超时** | `createIdleTimeoutManager` — 无活动自动退出 |
| **优雅关闭** | `gracefulShutdown` + `gracefulShutdownSync` — 清理资源 |
| **设置变更** | `settingsChangeDetector` + `applySettingsChange` — 实时响应 |
| **提示建议** | `tryGenerateSuggestion` — PromptSuggestion 投机执行 |

### `exit.ts` — Never 类型应用

```typescript
export function cliError(msg?: string): never {
  if (msg) console.error(msg)
  process.exit(1)
  return undefined as never  // TypeScript 控制流窄化
}
```

!!! tip "TypeScript 技巧"
    `never` 返回类型告诉编译器此函数永远不会正常返回，调用点之后的代码被视为不可达——避免了"变量可能未赋值"的误报。

## handlers/ — CLI 子命令

### `handlers/mcp.tsx` — MCP 管理（55KB）

实现 `claude mcp add/remove/list/get` 等子命令，是 MCP 服务器配置的 CLI 入口。

### `handlers/plugins.ts` — 插件管理（30KB）

实现 `claude plugins install/uninstall/list` 等子命令。

### `handlers/auth.ts` — 认证管理（11KB）

实现 `claude auth login/logout/status` 等子命令，处理 OAuth 令牌。

## 总结

`cli/` 是"面向用户的输出层"——`print.ts`（208KB/5595行）是非交互式渲染的巨型中枢，`structuredIO.ts` 处理 SDK JSON 通信，`handlers/` 实现 CLI 子命令。`print.ts` 之所以这么大，是因为它需要处理所有消息类型、所有输出格式、所有边缘情况的渲染逻辑。
