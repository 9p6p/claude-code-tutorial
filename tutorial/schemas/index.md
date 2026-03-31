# `schemas/` — Zod 校验 Schema

## 模块概述

集中管理所有 Zod 校验 Schema，为 API 响应、配置文件、消息格式等提供运行时类型验证。

## 关键 Schema

| Schema | 用途 |
|--------|------|
| `SettingsSchema` | 设置文件校验（`settings.json`） |
| `PermissionRuleSchema` | 权限规则校验 |
| `McpServerConfigSchema` | MCP 服务器配置校验 |
| `ToolInputSchema` | 工具输入参数校验 |
| `AgentDefinitionSchema` | Agent 定义校验 |
| `HookConfigSchema` | 用户自定义 Hook 校验 |

## 设计原则

- **Zod v4**：使用 `zod/v4` 最新版
- **懒加载**：`lazySchema(() => z.enum(...))` 延迟构建避免循环依赖
- **错误格式化**：`formatZodError(error, path)` 生成用户友好的错误消息
- **渐进验证**：`filterInvalidPermissionRules(data, path)` 一条坏规则不会拒绝整个文件

## 总结

`schemas/` 确保所有外部数据（配置文件、API 响应、用户输入）在进入系统前经过严格的类型校验，是"信任边界"上的守门人。
