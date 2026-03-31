# `keybindings/` — 键绑定系统

## 模块概述

完整的键绑定管理系统（14 个文件），支持默认绑定 + 用户自定义绑定 + Chord 序列 + 热更新。

## 文件清单

| 文件 | 说明 |
|------|------|
| `KeybindingProviderSetup.tsx` | 集成组件（加载 + 监听 + chord） |
| `KeybindingContext.tsx` | React 上下文 Provider |
| `defaultBindings.ts` | 默认键绑定定义 |
| `loadUserBindings.ts` | 加载用户自定义绑定 |
| `resolver.ts` | Chord 状态机解析器 |
| `parser.ts` | 按键序列解析 |
| `match.ts` | 按键匹配 |
| `validate.ts` | 配置校验 |
| `schema.ts` | 配置 schema |
| `reservedShortcuts.ts` | 保留快捷键 |
| `shortcutFormat.ts` | 快捷键格式化 |
| `template.ts` | 配置模板 |
| `useKeybinding.ts` | 注册键绑定 Hook |
| `useShortcutDisplay.ts` | 快捷键显示 Hook |

## Chord 支持

```typescript
const CHORD_TIMEOUT_MS = 1000
// Ctrl+K → 等待 1 秒 → Ctrl+C → 执行绑定
```

## 热更新

监听 `~/.claude/keybindings.json` 变更，自动重载。错误/警告通过通知系统显示。

## 总结

一个功能完整的键绑定框架——支持 Chord 多步按键、文件热更新、配置校验和分层覆盖。
