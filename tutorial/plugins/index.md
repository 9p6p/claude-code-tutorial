# `plugins/` — 内置插件系统

## 模块概述

`plugins/` 管理与 CLI 一起发布的**内置插件**。用户可通过 `/plugin` 命令启用/禁用。

## 核心文件：`builtinPlugins.ts`

### 插件注册表

```typescript
const BUILTIN_PLUGINS: Map<string, BuiltinPluginDefinition> = new Map()

export function registerBuiltinPlugin(definition: BuiltinPluginDefinition): void {
  BUILTIN_PLUGINS.set(definition.name, definition)
}
```

### 启用状态三级判断

```typescript
// 用户设置 > 插件默认 > true
const isEnabled =
  userSetting !== undefined
    ? userSetting === true
    : (definition.defaultEnabled ?? true)
```

### 插件 ID 格式

```
{name}@builtin    // 内置插件
{name}@{market}   // 市场插件
```

### 插件能力

每个内置插件可提供：
- **Skills**（技能命令）
- **Hooks**（事件钩子）
- **MCP Servers**（MCP 服务）

## 与 bundled skills 的区别

| 特性 | 内置插件 | 捆绑 Skills |
|------|---------|------------|
| 可管理 | 用户可启用/禁用 | 始终启用 |
| UI 展示 | 在 /plugin UI 中显示 | 不在 UI 中 |
| 组件 | Skills + Hooks + MCP | 仅 Skills |

## 总结

`builtinPlugins.ts` 实现了一个用户可控的插件注册表，通过 `isAvailable()` 门控和三级启用逻辑，在灵活性和安全性之间取得平衡。
