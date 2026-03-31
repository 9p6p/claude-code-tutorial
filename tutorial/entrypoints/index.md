# `entrypoints/` — 应用入口点

## 模块概述

Claude Code 的多种启动入口（6 个文件 + `sdk/` 子目录）。不同入口共享 `init.ts` 初始化逻辑。

## 文件清单

| 文件 | 说明 |
|------|------|
| `init.ts` | **核心初始化函数**（memoized，仅执行一次） |
| `cli.tsx` | CLI 主渲染入口 |
| `mcp.ts` | MCP 服务器入口 |
| `agentSdkTypes.ts` | Agent SDK 类型定义 |
| `sandboxTypes.ts` | 沙箱类型定义 |
| `sdk/coreSchemas.ts` | SDK 消息 Zod schemas（55KB） |
| `sdk/controlSchemas.ts` | 控制协议 schemas |

## `init.ts` — 启动链路

```
enableConfigs()
→ applySafeConfigEnvVars()
→ applyExtraCACerts()
→ configureGlobalMTLS()
→ configureGlobalAgents()
→ populateOAuthAccountInfo()
→ initializeTelemetry()    ← OpenTelemetry 动态 import (~400KB)
→ policyLimits
→ remoteManagedSettings
→ ensureScratchpadDir()
```

**Memoized** 保证全生命周期只执行一次。

## 三种运行模式

| 入口 | 说明 |
|------|------|
| `cli.tsx` | 交互式 REPL（默认模式） |
| `mcp.ts` | MCP 服务器模式（`claude mcp serve`） |
| `sdk/` | Agent SDK 模式（非交互式 `-p` 管道） |

## 总结

`entrypoints/` 是"大门"——不同入口（CLI/MCP/SDK）共享 `init.ts` 初始化逻辑（8 步启动链路），然后走各自的路径。SDK schemas 定义了完整的消息通信协议。
