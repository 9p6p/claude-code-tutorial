# `server/` — Direct Connect 服务器

## 模块概述

`server/` 实现了 Claude Code 的 **Direct Connect** 模式——用户可以启动一个本地或远程服务器，通过 WebSocket 与 Claude Code 交互。这是除 REPL 和 CCR 之外的第三种运行方式。

## 文件清单

| 文件 | 说明 |
|------|------|
| `createDirectConnectSession.ts` | 创建远程会话（HTTP POST） |
| `directConnectManager.ts` | WebSocket 会话管理器 |
| `types.ts` | 服务器配置和会话类型定义 |

## 会话创建

```typescript
export async function createDirectConnectSession({
  serverUrl, authToken, cwd, dangerouslySkipPermissions,
}): Promise<{ config: DirectConnectConfig; workDir?: string }> {
  const resp = await fetch(`${serverUrl}/sessions`, {
    method: 'POST',
    body: JSON.stringify({ cwd })
  })
  // Zod schema 验证响应
}
```

## 会话状态

```typescript
type SessionState = 'starting' | 'running' | 'detached' | 'stopping' | 'stopped'
```

支持会话分离（detach）和恢复——跨服务器重启持久化。

## 总结

`server/` 模块与 `remote/` 互补：`remote/` 连接 CCR 云端容器，`server/` 连接自托管服务器。两者共享类似的 WebSocket + 权限协议。
