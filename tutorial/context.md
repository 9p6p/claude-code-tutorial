# `context.ts` — 系统和用户上下文收集

## 文件概述

`context.ts` 负责收集**系统上下文**和**用户上下文**信息，这些信息最终会被注入到发送给 Claude API 的系统提示中。它回答一个关键问题：Claude 怎么知道用户的项目环境是什么样的？答案就在这个文件里。

## 关键代码解读

### 1. Git 状态收集

```typescript
export const getGitStatus = memoize(async (): Promise<string | null> => {
  const isGit = await getIsGit()
  if (!isGit) return null

  const [branch, mainBranch, status, log, userName] = await Promise.all([
    getBranch(),
    getDefaultBranch(),
    execFileNoThrow(gitExe(), ['status', '--short']),
    execFileNoThrow(gitExe(), ['log', '--oneline', '-n', '5']),
    execFileNoThrow(gitExe(), ['config', 'user.name']),
  ])
  // 组装上下文字符串
})
```

并行执行 5 个 git 命令收集：当前分支、默认分支、文件状态、最近 5 条提交、用户名。使用 `memoize` 缓存结果。

### 2. 系统提示注入

```typescript
let systemPromptInjection: string | null = null

export function setSystemPromptInjection(value: string | null): void {
  systemPromptInjection = value
  getUserContext.cache.clear?.()
  getSystemContext.cache.clear?.()
}
```

支持动态注入系统提示内容（仅内部调试用），注入后会清除上下文缓存。

### 3. CLAUDE.md 加载

```typescript
const claudeMds = getClaudeMds(
  getAdditionalDirectoriesForClaudeMd()
)
const memoryFiles = getMemoryFiles()
```

从项目根目录和配置目录加载 CLAUDE.md 文件和内存文件，这些是用户自定义的上下文补充。

### 4. getUserContext 和 getSystemContext

```typescript
export const getUserContext = memoize(async () => {
  // 收集 Git 信息、日期、平台信息等
  // 返回格式化的上下文字符串
})

export const getSystemContext = memoize(async () => {
  // 收集操作系统信息
  // 返回格式化的系统上下文
})
```

两个核心导出函数，分别提供用户级和系统级上下文。

## 核心函数列表

| 函数 | 说明 |
|------|------|
| `getGitStatus()` | 收集 Git 仓库状态信息 |
| `getUserContext()` | 用户上下文（Git + 日期 + CLAUDE.md） |
| `getSystemContext()` | 系统上下文（OS + 平台信息） |
| `getSystemPromptInjection()` | 获取调试注入内容 |
| `setSystemPromptInjection()` | 设置调试注入内容 |

## 与其他模块的关系

```
context.ts
  ├── utils/git.ts          → Git 操作
  ├── utils/claudemd.ts     → CLAUDE.md 加载
  ├── bootstrap/state.ts    → 全局状态
  ├── utils/execFileNoThrow → 安全执行子进程
  └── main.tsx              → 被主入口调用
```

## 总结

`context.ts` 是"让 Claude 了解你的环境"的关键桥梁。它通过并行的 Git 命令和配置文件读取，为每次 API 调用构建精确的上下文描述。`memoize` 缓存确保这些信息只在首次请求时计算。
