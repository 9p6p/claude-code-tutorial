# `remote/` — 远程会话管理

## 模块概述

`remote/` 模块处理 Claude Code 与**远程 CCR（Claude Code Remote）容器**之间的通信。当用户使用 `--remote` 模式时，AI 在远程容器中运行，本地 CLI 作为"瘦客户端"显示结果和处理权限请求。

## 文件清单

| 文件 | 说明 |
|------|------|
| `RemoteSessionManager.ts` | 远程会话管理器（WebSocket + HTTP） |
| `SessionsWebSocket.ts` | WebSocket 客户端（自动重连、ping keepalive） |
| `remotePermissionBridge.ts` | 远程权限请求的消息桥接 |
| `sdkMessageAdapter.ts` | SDK 消息格式 → REPL 内部格式转换 |

## 核心架构

```
远程 CCR 容器（执行 AI + 工具）
    ↕ WebSocket (SDK 消息流)
    ↕ HTTP POST (用户消息)
SessionsWebSocket.ts
    ↓
RemoteSessionManager.ts
    ↓ 消息路由
    ├─ SDKMessage → sdkMessageAdapter → REPL 渲染
    └─ ControlRequest (权限) → remotePermissionBridge → 本地权限 UI
```

## 关键设计

- **双通道通信**：WebSocket 接收消息流，HTTP POST 发送用户输入
- **权限桥接**：远程工具调用需要本地用户确认权限
- **自动重连**：指数退避 + 4001（compaction 期间临时错误）有限重试
- **Bun/Node 双运行时**：根据环境选择不同的 WebSocket 实现

## 总结

`remote/` 模块让 Claude Code 变成"瘦客户端"——AI 在云端执行，本地只负责显示和权限管理。
