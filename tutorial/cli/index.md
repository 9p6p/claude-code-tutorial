# `cli/` — CLI 输出与通信

## 模块概述

CLI 的输出渲染和 I/O 通信层（8 个文件 + handlers/ 和 transports/ 子目录）。

## 文件清单

| 文件 | 大小 | 说明 |
|------|------|------|
| `print.ts` | **208KB** | 消息→终端渲染引擎（巨型文件！） |
| `structuredIO.ts` | 28KB | SDK 结构化 JSON I/O |
| `update.ts` | 14KB | 自动更新 |
| `remoteIO.ts` | 10KB | 远程 I/O |
| `exit.ts` | 1.3KB | CLI 退出辅助函数 |
| `ndjsonSafeStringify.ts` | 1.4KB | NDJSON 安全序列化 |
| `handlers/mcp.tsx` | 55KB | MCP 管理子命令 |
| `handlers/plugins.ts` | 30KB | 插件管理子命令 |
| `handlers/auth.ts` | 11KB | 认证子命令 |

## exit.ts — 最简的 Never 类型应用

```typescript
export function cliError(msg?: string): never {
  if (msg) console.error(msg)
  process.exit(1)
  return undefined as never
}
```

`: never` 返回类型让 TypeScript 在调用点后自动窄化控制流。

## 总结

`cli/` 是"面向用户的输出层"。`print.ts`（208KB）是项目中数一数二的大文件，负责将所有消息类型渲染为终端可读格式。
