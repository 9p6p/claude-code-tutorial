# `types/` — 核心类型定义

## 模块概述

`types/` 是纯类型定义库（8 个文件 + generated/ 子目录），专门用于打破导入循环。

## 文件清单

| 文件 | 说明 |
|------|------|
| `permissions.ts` | 权限系统核心类型 |
| `hooks.ts` | Hook 系统类型和 Schema |
| `command.ts` | 命令类型定义 |
| `plugin.ts` | 插件类型定义 |
| `textInputTypes.ts` | 文本输入类型 |
| `logs.ts` | 日志类型 |
| `ids.ts` | 品牌化 ID 类型 |
| `generated/` | Protobuf 生成的事件类型 |

## 品牌化类型

```typescript
export type SessionId = string & { readonly __brand: 'SessionId' }
export type AgentId = string & { readonly __brand: 'AgentId' }
```

编译时防止 ID 混用——传入 SessionId 的地方不能传 AgentId。

## 权限类型

```typescript
type PermissionMode = 'acceptEdits' | 'bypassPermissions' | 'default' | 'plan' | 'auto' | 'bubble'
type PermissionBehavior = 'allow' | 'deny' | 'ask'
type PermissionUpdate = 
  | { type: 'addRules'; ... }
  | { type: 'setMode'; ... }
  | { type: 'addDirectories'; ... }
```

## 总结

`types/` 是循环依赖的"解药"——纯类型文件无运行时依赖，多个模块可以安全地从这里导入。
