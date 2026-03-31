# `dialogLaunchers.tsx` — 对话框启动器

## 文件概述

`dialogLaunchers.tsx` 包含一组**对话框启动函数**，用于在 `main.tsx` 中展示各种一次性对话框（设置验证、会话恢复、Agent 快照更新等）。每个启动器使用动态导入，只在需要时加载对应组件。

## 关键代码解读

### 1. 启动器模式

```typescript
export async function launchSnapshotUpdateDialog(
  root: Root, 
  props: { agentType: string; scope: AgentMemoryScope; ... }
): Promise<'merge' | 'keep' | 'replace'> {
  const { SnapshotUpdateDialog } = await import(
    './components/agents/SnapshotUpdateDialog.js'
  )
  return showSetupDialog<'merge' | 'keep' | 'replace'>(
    root, done => <SnapshotUpdateDialog {...props} onComplete={done} />
  )
}
```

每个启动器遵循相同模式：
1. 动态导入组件
2. 调用 `showSetupDialog` 渲染到终端
3. 返回 Promise，等待用户交互完成

### 2. 可用的对话框

| 启动器 | 用途 |
|--------|------|
| `launchSnapshotUpdateDialog` | Agent 内存快照更新选择 |
| `launchInvalidSettingsDialog` | 设置验证错误提示 |
| `launchAssistantSessionChooser` | Bridge 会话选择 |
| `launchResumeChooser` | 恢复已有对话 |
| `launchTeleportResumeWrapper` | 远程传送恢复 |
| `launchTeleportRepoMismatchDialog` | 仓库不匹配警告 |
| `launchAssistantInstallWizard` | Assistant 安装向导 |

## 设计要点

- **零运行时开销**：动态导入确保不用的对话框不占内存
- **类型安全**：每个启动器有明确的输入和返回类型
- **回调桥接**：将 React 组件的回调模式转换为 Promise 模式

## 与其他模块的关系

```
dialogLaunchers.tsx
  ├── interactiveHelpers.tsx → 提供 showSetupDialog 基础设施
  ├── components/           → 各个对话框 UI 组件
  ├── ink.ts                → TUI 渲染
  └── main.tsx              → 唯一的调用方
```

## 总结

`dialogLaunchers.tsx` 是 `main.tsx` 的"JSX 提取"产物——将内联的 React 渲染代码提取到独立函数中，提高 main.tsx 的可读性和可维护性。
