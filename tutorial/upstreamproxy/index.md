# `upstreamproxy/` — CCR 上游代理

## 模块概述

`upstreamproxy/` 实现了 CCR（Claude Code Remote）容器内的 **HTTPS 代理中继**。当 Claude Code 在远程容器中运行时，某些外部 API 请求需要通过代理注入组织凭证（如 Datadog API Key）。

## 文件清单

| 文件 | 大小 | 说明 |
|------|------|------|
| `relay.ts` | 14.6KB | CONNECT-over-WebSocket 中继 |
| `upstreamproxy.ts` | 9.6KB | 容器端初始化和环境变量注入 |

## 工作原理

```
子进程 (curl/gh/python)
  → HTTPS_PROXY=127.0.0.1:PORT
  → 本地 TCP (relay.ts 监听)
  → HTTP CONNECT → WebSocket 封装
  → CCR 服务端（解封装 → MITM TLS → 注入凭证 → 转发）
  → 真实上游服务器
```

## 安全设计

1. **prctl(PR_SET_DUMPABLE, 0)**：阻止同 UID 进程 ptrace 读取堆内存中的 token
2. **Token 文件即删**：读取后立即删除，token 仅存在于进程堆中
3. **Fail Open**：任何错误都降级禁用代理，不影响会话
4. **手工 Protobuf**：单字段消息手写编解码，避免依赖 protobufjs

## NO_PROXY 列表

```typescript
const NO_PROXY_LIST = [
  'localhost', '127.0.0.1',
  'anthropic.com', '*.anthropic.com',
  'github.com', '*.github.com',
  'registry.npmjs.org', 'pypi.org',
  // ...
]
```

关键服务绕过代理，避免 MITM 破坏 TLS。

## 总结

这是一个高度安全敏感的模块，展示了企业级代理隧道的实现：双运行时支持（Bun/Node）、手工 Protobuf、安全的 token 管理、容错的 fail-open 设计。
