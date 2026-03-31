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

## 关键文件

### `print.ts` — 208KB 渲染引擎

项目中数一数二的大文件，将所有消息类型渲染为非交互式终端输出（`-p` 管道模式和 `--output-format text` 的核心）。

### `exit.ts` — Never 类型应用

```typescript
export function cliError(msg?: string): never {
  if (msg) console.error(msg)
  process.exit(1)
  return undefined as never  // TypeScript 控制流窄化
}
```

## 总结

`cli/` 是"面向用户的输出层"——`print.ts`（208KB）负责非交互式渲染，`structuredIO.ts` 处理 SDK JSON 通信，`handlers/` 实现命令行子命令。
