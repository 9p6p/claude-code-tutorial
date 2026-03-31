# `memdir/` — 持久化记忆系统

## 模块概述

`memdir/` 管理 AI 助手的**持久化记忆**——MEMORY.md 文件和自动记忆目录（8 个文件）。这是 Claude Code "记住"用户偏好和项目知识的核心机制。

## 文件清单

| 文件 | 说明 |
|------|------|
| `memdir.ts` | MEMORY.md 加载、截断、提示构建 |
| `paths.ts` | 路径计算和安全校验 |
| `memoryTypes.ts` | 记忆类型的提示词模板 |
| `memoryAge.ts` | 记忆时效管理 |
| `memoryScan.ts` | 记忆扫描 |
| `findRelevantMemories.ts` | 相关记忆检索 |
| `teamMemPaths.ts` | 团队记忆路径 |
| `teamMemPrompts.ts` | 团队记忆提示 |

## 关键设计

### MEMORY.md 限制

```typescript
export const MAX_ENTRYPOINT_LINES = 200
export const MAX_ENTRYPOINT_BYTES = 25_000
```

### 安全校验

```typescript
function validateMemoryPath(raw: string | undefined): string | undefined {
  // 拒绝：相对路径、根路径、UNC 路径、null 字节
  // 防止写入 ~/.ssh 等敏感目录
}
```

### 配置优先级链

```
env var > settings.json > 默认路径
(CLAUDE_CODE_DISABLE_AUTO_MEMORY → autoMemoryEnabled → enabled)
```

## 总结

`memdir/` 是 Claude Code 的"长期记忆"——通过安全的路径校验和多层配置，在便利性和安全性之间取得平衡。
