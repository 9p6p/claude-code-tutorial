# `schemas/` — Zod 校验 Schema

## 模块概述

`schemas/` 目录包含从其他模块提取出来的 Zod schema 定义，目的是**打破循环依赖**。

## 文件：`hooks.ts`

定义了 Claude Code **Hook 系统**的完整 schema。Hook 允许在特定事件（工具调用前后、会话开始等）执行自定义逻辑。

### 四种 Hook 类型

```typescript
// 1. Shell 命令 Hook
{ type: 'command', command: 'npm test', timeout: 30 }

// 2. LLM 提示 Hook  
{ type: 'prompt', prompt: 'Review this code change', model: 'claude-sonnet-4-6' }

// 3. HTTP Hook
{ type: 'http', url: 'https://webhook.example.com', headers: {...} }

// 4. Agent 验证器 Hook
{ type: 'agent', prompt: 'Verify unit tests passed' }
```

### Matcher 系统

```typescript
{
  matcher: 'Bash(git *)',  // 匹配 git 命令
  hooks: [
    { type: 'command', command: 'echo "git operation detected"' }
  ]
}
```

使用权限规则语法过滤何时触发 Hook。

## 总结

`schemas/hooks.ts` 展示了如何用 Zod + 判别联合（Discriminated Union）构建类型安全的配置系统。`lazySchema()` 技巧避免了循环依赖。
