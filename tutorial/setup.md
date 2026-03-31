# `setup.ts` — 环境初始化

## 文件概述

`setup.ts` 负责 Claude Code 启动时的**环境准备工作**。在 `main.tsx` 解析完 CLI 参数后、启动 REPL 之前，`setup()` 函数会被调用来完成工作目录设置、Git 检测、会话初始化、Worktree 创建等关键步骤。

## 关键代码解读

### 1. setup() 主函数签名

```typescript
export async function setup(
  cwd: string,
  permissionMode: PermissionMode,
  allowDangerouslySkipPermissions: boolean,
  worktreeEnabled: boolean,
  worktreeName: string | undefined,
  tmuxEnabled: boolean,
  customSessionId?: string | null,
  worktreePRNumber?: number,
  messagingSocketPath?: string,
): Promise<void>
```

参数丰富，涵盖了所有可能的启动配置选项。

### 2. Node.js 版本检查

```typescript
const nodeVersion = process.version.match(/^v(\d+)\./)?.[1]
if (!nodeVersion || parseInt(nodeVersion) < 18) {
  console.error(chalk.bold.red(
    'Error: Claude Code requires Node.js version 18 or higher.'
  ))
  process.exit(1)
}
```

启动时首先检查 Node.js 版本是否满足最低要求。

### 3. 项目根目录发现

```typescript
const gitRoot = await findCanonicalGitRoot()
if (gitRoot) {
  setProjectRoot(gitRoot)
} else {
  setProjectRoot(cwd)
}
```

通过 Git 根目录发现机制确定项目根目录，如果不在 Git 仓库中则使用当前目录。

### 4. Worktree 模式

```typescript
if (worktreeEnabled) {
  const worktreePath = await createWorktreeForSession(
    worktreeName || worktreeBranchName(getSessionId())
  )
  setCwd(worktreePath)
  
  if (tmuxEnabled) {
    await createTmuxSessionForWorktree(
      generateTmuxSessionName(), worktreePath
    )
  }
}
```

Worktree 模式允许 Claude Code 在独立的 Git worktree 中工作，避免干扰用户的主工作区。还可以结合 tmux 创建独立终端会话。

### 5. 会话初始化

```typescript
if (customSessionId) {
  switchSession(asSessionId(customSessionId))
} else {
  switchSession(asSessionId(randomUUID()))
}
await initSessionMemory()
```

创建或恢复会话 ID，并初始化会话级别的内存系统。

## 核心函数列表

| 函数 | 说明 |
|------|------|
| `setup()` | 主初始化函数 |
| `findCanonicalGitRoot()` | 查找 Git 根目录 |
| `createWorktreeForSession()` | 创建 Git Worktree |
| `createTmuxSessionForWorktree()` | 创建 tmux 会话 |
| `initSessionMemory()` | 初始化会话内存 |
| `lockCurrentVersion()` | 锁定当前版本（防止更新） |

## 与其他模块的关系

```
setup.ts
  ├── bootstrap/state.ts      → 设置全局状态
  ├── utils/git.ts            → Git 操作
  ├── utils/worktree.ts       → Worktree 管理
  ├── utils/Shell.ts          → 设置工作目录
  ├── services/SessionMemory/ → 会话内存
  ├── utils/config.ts         → 配置读取
  └── main.tsx                → 被主入口调用
```

## 总结

`setup.ts` 是"开工前的准备工作"——确保运行环境、工作目录、会话状态都正确就绪。核心特点：
- **Worktree 隔离**：支持在独立工作树中工作
- **Tmux 集成**：可自动创建终端会话
- **版本锁定**：防止会话期间的自动更新
