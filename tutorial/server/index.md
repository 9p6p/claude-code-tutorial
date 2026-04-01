# `server/` — Direct Connect 服务器

## 模块概述

`server/` 实现了 Claude Code 的 **Direct Connect** 模式——用户可以连接到自托管服务器，通过 WebSocket 与 Claude Code 交互。这是除 REPL 和 CCR 之外的第三种运行方式。

## 文件清单

| 文件 | 说明 |
|------|------|
| `createDirectConnectSession.ts` | 创建远程会话（HTTP POST） |
| `directConnectManager.ts` | WebSocket 会话管理器 |
| `types.ts` | 服务器配置和会话类型定义 |

## 与 `remote/` 的对比

| 维度 | `remote/`（CCR） | `server/`（Direct Connect） |
|------|-----------------|--------------------------|
| 目标 | Anthropic 云端容器 | 自托管服务器 |
| 认证 | OAuth + 消息级认证 | Bearer Token（headers） |
| 重连 | 自动重连（5次） | **无自动重连** |
| 复杂度 | 高（compaction 处理） | 低（更简单的协议） |
| WebSocket 库 | Bun + ws 双支持 | 仅 Bun 原生 |

## 会话创建

```typescript
export async function createDirectConnectSession({
  serverUrl, authToken, cwd, dangerouslySkipPermissions,
}): Promise<{ config: DirectConnectConfig; workDir?: string }> {
  const resp = await fetch(`${serverUrl}/sessions`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      ...(authToken ? { Authorization: `Bearer ${authToken}` } : {}),
    },
    body: JSON.stringify({ cwd }),
  })
  // Zod schema 验证响应
}
```

## `DirectConnectSessionManager` — WebSocket 管理

```typescript
type DirectConnectConfig = {
  serverUrl: string
  sessionId: string
  wsUrl: string        // WebSocket 地址
  authToken?: string
}
```

### 消息处理

| 消息类型 | 处理方式 |
|----------|---------|
| `control_request.can_use_tool` | → 转发权限请求 |
| 未知 `control_request` 子类型 | → 发送错误响应（**防止服务端挂起**） |
| `control_response` | → 过滤（内部消息） |
| `keep_alive` | → 过滤 |
| `streamlined_*` | → 过滤 |
| 其他 `SDKMessage` | → 转发给 `onMessage` |

!!! warning "防挂起设计"
    对于不认识的 `control_request` 子类型，主动发送错误响应——避免服务端无限等待一个永远不会来的回复。

### 通信 API

```typescript
// 发送用户消息
sendMessage(content: RemoteMessageContent): boolean

// 响应权限请求
respondToPermissionRequest(requestId, result: RemotePermissionResponse): void

// 发送中断信号
sendInterrupt(): void
```

### 权限响应类型

```typescript
type RemotePermissionResponse =
  | { behavior: 'allow'; updatedInput: Record<string, unknown> }
  | { behavior: 'deny'; message: string }
```

## 会话状态

```typescript
type SessionState = 'starting' | 'running' | 'detached' | 'stopping' | 'stopped'
```

支持会话分离（detach）和恢复——跨服务器重启持久化。

## 总结

`server/` 模块与 `remote/` 互补：`remote/` 连接 Anthropic CCR 云端容器，`server/` 连接自托管服务器。`DirectConnectSessionManager` 比 `RemoteSessionManager` 更简单——无自动重连、无 compaction 处理、无 OAuth 消息级认证——但共享相同的 WebSocket + 权限协议框架。
