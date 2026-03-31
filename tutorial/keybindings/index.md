# `keybindings/` — 键绑定系统

## 模块概述

可配置的快捷键系统，支持用户自定义按键映射和多上下文快捷键注册。

## 架构

```
KeybindingProvider (顶层 Context)
  └── useKeybinding('chat:cancel', handler, { context, isActive })
       └── KeybindingResolver (按优先级匹配)
            └── 用户自定义 > 内置默认
```

## 快捷键分类

| 上下文 | 示例快捷键 |
|--------|-----------|
| Chat | `Escape`/`Ctrl+C` (取消), `Ctrl+O` (转录) |
| Transcript | `j/k` (上下导航), `/` (搜索) |
| Confirmation | `Escape`/`n` (取消), `Enter` (确认) |
| Help | `Escape` (关闭) |

## 核心组件

| 文件 | 说明 |
|------|------|
| `KeybindingContext.ts` | React Context 提供快捷键解析器 |
| `KeybindingProviderSetup.tsx` | 初始化 + 加载用户自定义配置 |
| `resolver.ts` | 匹配逻辑（支持修饰键组合、多键序列） |
| `useKeybinding.ts` | Hook：注册快捷键处理器 |
| `shortcutFormat.ts` | 格式化显示（`⌘C`、`Ctrl+C`） |
| `types.ts` | `ParsedKeystroke`（key + ctrl/shift/alt/meta/super） |
| `parser.ts` | 解析字符串 `"ctrl+shift+t"` → `ParsedKeystroke` |

## 总结

`keybindings/` 让用户可以通过配置文件自定义所有快捷键，同时提供声明式的 `useKeybinding` Hook 让组件轻松注册快捷键处理器。
