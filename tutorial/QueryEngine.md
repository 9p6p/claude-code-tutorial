# `QueryEngine.ts` — LLM 查询引擎

## 文件概述

`QueryEngine.ts` 是 Claude Code 的**LLM 查询引擎核心**，大约 1,300 行代码。它封装了与 Claude API 的完整交互循环，包括发送请求、处理流式响应、执行工具调用、管理重试和错误处理。可以把它理解为 Claude Code 的"大脑"——它驱动着每一轮 AI 对话。

## 关键代码解读

### 1. 核心导入

```typescript
import { query } from './query.js'
import { accumulateUsage, updateUsage } from './services/api/claude.js'
import { loadMemoryPrompt } from './memdir/memdir.js'
import { processUserInput } from './utils/processUserInput/processUserInput.js'
import { fetchSystemPromptParts } from './utils/queryContext.js'
```

QueryEngine 组合了多个子系统：查询执行（query.ts）、使用量统计、内存系统、用户输入处理、系统提示构建。

### 2. 查询循环模式

```
用户输入 → processUserInput() 
        → fetchSystemPromptParts() 
        → query() [发送到 Claude API]
        → 流式响应处理
        → 工具调用检测
        → 执行工具 → 结果回注 → 再次 query()
        → 直到无工具调用 → 输出最终回答
```

这是一个**递归/迭代循环**：Claude 的回答可能包含工具调用，工具执行后的结果会作为新的消息追加到对话中，然后再次发送给 Claude，直到得到纯文本回答。

### 3. 系统提示构建

```typescript
const systemPromptParts = await fetchSystemPromptParts({
  cwd: getCwd(),
  tools,
  commands,
  mcpClients,
  // ...
})
```

系统提示不是一个静态字符串，而是由多个部分动态组装：
- 基础指令
- 工具描述
- 当前上下文（Git 状态、文件结构）
- CLAUDE.md 内容
- 内存文件

### 4. 会话记录与恢复

```typescript
recordTranscript(messages)
flushSessionStorage()
```

每次查询完成后，对话记录会被写入磁盘，支持会话恢复和审计。

### 5. Fast Mode 和 Advisor

```typescript
const fastModeState = getFastModeState()
if (fastModeState.enabled) {
  // 使用更快的模型处理简单请求
}
```

QueryEngine 支持"快速模式"——简单请求使用轻量模型，复杂请求使用完整模型。

### 6. SDK 模式支持

```typescript
// SDK 模式：不启动 REPL，直接返回结构化结果
if (sdkMode) {
  return { messages, usage, cost }
}
```

QueryEngine 同时服务于交互式 REPL 和编程式 SDK 两种使用方式。

## 核心功能汇总

| 功能 | 说明 |
|------|------|
| 查询循环 | 用户输入 → LLM 响应 → 工具执行 → 循环 |
| 系统提示 | 动态组装多部分系统提示 |
| 成本跟踪 | 每次 API 调用记录 token 使用量 |
| 会话记录 | 持久化对话历史到磁盘 |
| 错误恢复 | 可重试的 API 错误自动重试 |
| 多模式 | 支持 REPL、SDK、打印等模式 |
| 文件快照 | 记录工具调用前后的文件状态 |

## 与其他模块的关系

```
QueryEngine.ts（查询引擎核心）
  ├── query.ts               → 底层 API 调用
  ├── services/api/claude.ts → Claude API 客户端
  ├── Tool.ts                → 工具类型和上下文
  ├── cost-tracker.ts        → 成本统计
  ├── memdir/                → 内存系统
  ├── utils/queryContext.ts  → 系统提示构建
  ├── utils/processUserInput → 用户输入预处理
  ├── utils/sessionStorage   → 会话持久化
  └── screens/REPL.tsx       → REPL 界面集成
```

## 总结

`QueryEngine.ts` 是整个 Claude Code 的"心脏"。核心设计亮点：
- **工具执行循环**：自动递归直到完成任务
- **动态系统提示**：根据上下文实时构建
- **多模式适配**：同一引擎服务 REPL 和 SDK
- **弹性设计**：内置重试和错误分类机制
