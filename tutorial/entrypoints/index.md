# `entrypoints/` — 应用入口点

## 模块概述

`entrypoints/` 包含 Claude Code 的多种启动入口（6 个文件 + sdk/ 子目录）。`init.ts` 是核心初始化函数，`cli.tsx` 是 CLI 主入口。

## 文件清单

| 文件 | 说明 |
|------|------|
| `init.ts` | 核心初始化函数（memoized，仅执行一次） |
| `cli.tsx` | CLI 主渲染入口 |
| `mcp.ts` | MCP 服务器入口 |
| `agentSdkTypes.ts` | Agent SDK 类型定义 |
| `sandboxTypes.ts` | 沙箱类型定义 |
| `sdk/coreSchemas.ts` | SDK 消息 Zod schemas（55KB） |
| `sdk/controlSchemas.ts` | 控制协议 schemas |

## init.ts — 启动链路

```
enableConfigs() → applySafeConfigEnvVars() → applyExtraCACerts() 
→ configureGlobalMTLS() → configureGlobalAgents() 
→ populateOAuthAccountInfo() → initializeTelemetry()
→ policyLimits → remoteManagedSettings → ensureScratchpadDir()
```

Memoized 保证全生命周期只执行一次。OpenTelemetry 通过动态 `import()` 延迟加载（~400KB）。

## 总结

`entrypoints/` 是"大门"——不同入口（CLI/MCP/SDK）共享 `init.ts` 初始化逻辑，然后走各自的路径。
