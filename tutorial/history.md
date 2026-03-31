# `history.ts` — 命令历史管理

## 文件概述

`history.ts` 管理 Claude Code 的**命令历史记录**。它负责记录用户的每次输入，处理粘贴内容（文本和图片）的存储与还原，以及提供历史浏览功能。类似 shell 的历史记录但更强大——它还能处理图片和多行文本粘贴。

## 关键代码解读

### 1. 粘贴内容引用系统

```typescript
export function formatPastedTextRef(id: number, numLines: number): string {
  if (numLines === 0) return `[Pasted text #${id}]`
  return `[Pasted text #${id} +${numLines} lines]`
}

export function formatImageRef(id: number): string {
  return `[Image #${id}]`
}
```

用户粘贴的大段文本或图片会被替换为引用标记（如 `[Pasted text #1 +10 lines]`），实际内容存储在外部 paste store 中。这避免了历史记录文件过大。

### 2. 引用解析

```typescript
export function parseReferences(
  input: string,
): Array<{ id: number; match: string; index: number }> {
  const referencePattern =
    /\[(Pasted text|Image|\.\.\.Truncated text) #(\d+)(?: \+\d+ lines)?(\.)*\]/g
  // ...
}
```

从用户输入中解析出所有粘贴引用，用于后续内容还原。

### 3. 大文件存储策略

```typescript
type StoredPastedContent = {
  id: number
  type: 'text' | 'image'
  content?: string         // 小内容直接内联
  contentHash?: string     // 大内容存储 hash 引用
  mediaType?: string
  filename?: string
}
```

小于 `MAX_PASTED_CONTENT_LENGTH`(1024) 的内容直接内联存储；大内容通过 hash 存储在外部 paste store 中。

### 4. 历史文件读写

```typescript
const MAX_HISTORY_ITEMS = 100

// 使用文件锁保证并发安全
const lock = await lockfile(historyPath)
await appendFile(historyPath, JSON.stringify(entry))
```

历史记录以 JSON Lines 格式追加到文件末尾，使用文件锁处理并发访问。最多保留 100 条记录。

## 核心函数列表

| 函数 | 说明 |
|------|------|
| `addToHistory()` | 添加新的历史记录 |
| `getHistory()` | 获取历史记录列表 |
| `parseReferences()` | 解析输入中的粘贴引用 |
| `formatPastedTextRef()` | 格式化文本粘贴引用 |
| `formatImageRef()` | 格式化图片引用 |
| `getPastedTextRefNumLines()` | 计算粘贴文本的行数 |

## 与其他模块的关系

```
history.ts
  ├── utils/pasteStore.ts     → 大型粘贴内容的外部存储
  ├── utils/lockfile.ts       → 文件锁
  ├── utils/fsOperations.ts   → 文件系统操作
  ├── bootstrap/state.ts      → 会话 ID
  └── main.tsx                → 被主入口调用
```

## 总结

`history.ts` 实现了一个"增强版 shell 历史"：不仅记录文本命令，还能处理图片和大段粘贴内容。核心设计：
- **引用系统**：大内容用标记引用，保持历史文件轻量
- **外部存储**：通过 paste store + hash 管理大型内容
- **并发安全**：文件锁保护历史文件
