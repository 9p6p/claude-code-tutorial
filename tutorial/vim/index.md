# `vim/` — Vim 输入模式

## 模块概述

`vim/` 模块实现了 Claude Code 终端中的完整 **Vim 输入模式**——包括 INSERT/NORMAL 模式切换、移动命令、操作符、文本对象、查找、dot repeat 等。这是一个精心设计的有限状态机。

## 文件清单

| 文件 | 大小 | 说明 |
|------|------|------|
| `types.ts` | 6.2KB | 状态机类型定义（状态图即文档） |
| `motions.ts` | 1.9KB | 纯函数：移动命令 → 目标位置 |
| `textObjects.ts` | 4.9KB | 文本对象边界查找（iw/aw/i"/a(） |
| `operators.ts` | 15.6KB | 操作符执行（delete/change/yank/paste 等） |
| `transitions.ts` | 12.1KB | 状态机转换表 |

## 架构分层

```
types.ts (类型 + 常量)
    ↓
motions.ts (移动)  textObjects.ts (文本对象)
    ↓                    ↓
operators.ts (操作符执行)
    ↓
transitions.ts (状态机入口)
```

## 关键设计

### 状态机

```typescript
export type VimState =
  | { mode: 'INSERT'; insertedText: string }
  | { mode: 'NORMAL'; command: CommandState }

export type CommandState =
  | { type: 'idle' }
  | { type: 'count'; digits: string }
  | { type: 'operator'; op: Operator; count: number }
  | { type: 'find'; find: FindType; count: number }
  | { type: 'replace'; count: number }
  // ... 共 11 种状态
```

### 纯函数设计

所有核心逻辑都是纯函数，无副作用：
- `resolveMotion(key, cursor, count)` → 新光标位置
- `findTextObject(text, offset, type, isInner)` → 范围
- `transition(state, input, ctx)` → 新状态 + 执行函数

### Grapheme-Safe

正确处理 Unicode grapheme clusters（如 emoji），不会在字符中间断裂。

## 总结

一个教科书级的状态机实现——类型即文档、纯函数、分层清晰。
