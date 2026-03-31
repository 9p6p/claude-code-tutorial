# `query.ts` — API 查询底层实现

## 文件概述

`query.ts` 是 Claude Code 与 Claude API 交互的**底层实现层**，约 1,730 行代码。如果说 `QueryEngine.ts` 是"大脑"，那 `query.ts` 就是"神经网络"——它处理消息格式化、流式传输、工具调用执行、上下文压缩等底层细节。

## 关键功能

### 1. 消息发送与流式处理

`query()` 函数是核心入口，负责：
1. 将内部消息格式转换为 Claude API 格式
2. 通过流式 API 发送请求
3. 逐 chunk 解析响应
4. 检测和分发工具调用
5. 收集使用量统计

### 2. 工具调用执行

当 Claude 回复中包含工具调用（`tool_use` content block）时，`query.ts` 会：
1. 从工具注册表中查找对应工具
2. 验证工具参数
3. 检查权限
4. 执行工具并收集结果
5. 将结果作为 `tool_result` 追加到消息中

### 3. 上下文压缩（Compact）

当对话历史过长接近上下文窗口限制时，会触发自动压缩：
```
原始消息 [msg1, msg2, ..., msg100]
  → 压缩请求 → Claude 生成摘要
  → 压缩后 [summary, msg99, msg100]
```

### 4. 消息格式化

将内部的富消息格式（包含 UI 信息、工具进度等）转换为 Claude API 接受的标准格式。

### 5. 权限检查

在执行每个工具调用前检查权限配置：
- `alwaysAllow`：直接执行
- `alwaysDeny`：拒绝并返回错误
- `alwaysAsk`：提示用户确认

## 核心函数

| 函数 | 说明 |
|------|------|
| `query()` | 主查询函数 |
| `processToolCalls()` | 处理批量工具调用 |
| `executeToolCall()` | 执行单个工具调用 |
| `normalizeMessagesForAPI()` | 消息格式转换 |
| `handleCompact()` | 上下文压缩 |
| `handleRetry()` | API 错误重试 |

## 与其他模块的关系

```
query.ts（底层 API 层）
  ├── services/api/claude.ts → API 客户端
  ├── Tool.ts                → 工具接口
  ├── tools.ts               → 工具注册表
  ├── hooks/useCanUseTool    → 权限检查
  ├── QueryEngine.ts         → 高层查询引擎（调用方）
  └── utils/messages.ts      → 消息工具函数
```

## 总结

`query.ts` 是 Claude Code 与 AI 模型交互的"管道层"。它处理了所有协议细节，让上层的 QueryEngine 可以专注于业务逻辑。核心挑战在于正确处理流式响应中的多工具并行调用和上下文窗口管理。
