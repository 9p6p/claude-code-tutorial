# `projectOnboardingState.ts` — 项目引导状态

## 文件概述

`projectOnboardingState.ts` 管理**单个项目的引导流程状态**——追踪用户是否已完成项目级别的设置步骤（如创建 CLAUDE.md 文件）。

## 关键代码解读

### 1. 引导步骤定义

```typescript
export type Step = {
  key: string          // 步骤唯一标识
  text: string         // 显示给用户的描述
  isComplete: boolean  // 是否已完成
  isCompletable: boolean // 是否可以完成
  isEnabled: boolean   // 是否当前启用
}

export function getSteps(): Step[] {
  const hasClaudeMd = existsSync(join(getCwd(), 'CLAUDE.md'))
  const isWorkspaceDirEmpty = isDirEmpty(getCwd())

  return [
    {
      key: 'workspace',
      text: 'Ask Claude to create a new app or clone a repository',
      isComplete: false,
      isCompletable: true,
      isEnabled: isWorkspaceDirEmpty,
    },
    {
      key: 'claudemd',
      text: 'Run /init to create a CLAUDE.md file',
      isComplete: hasClaudeMd,
      isCompletable: true,
      isEnabled: !isWorkspaceDirEmpty,
    },
  ]
}
```

两个步骤：
- 空目录：提示创建项目或克隆仓库
- 有项目：提示创建 CLAUDE.md 配置文件

### 2. 完成检查

```typescript
export function isProjectOnboardingComplete(): boolean {
  return getSteps()
    .filter(({ isCompletable, isEnabled }) => isCompletable && isEnabled)
    .every(({ isComplete }) => isComplete)
}
```

只有所有启用且可完成的步骤都完成了，引导才算完成。

### 3. 自动标记完成

```typescript
export function maybeMarkProjectOnboardingComplete(): void {
  if (getCurrentProjectConfig().hasCompletedProjectOnboarding) return
  
  if (isProjectOnboardingComplete()) {
    saveCurrentProjectConfig(current => ({
      ...current,
      hasCompletedProjectOnboarding: true,
    }))
  }
}
```

在每次用户提交提示时调用，检查是否满足完成条件。使用配置缓存避免不必要的文件系统检查。

## 与其他模块的关系

```
projectOnboardingState.ts
  ├── utils/config.ts    → 项目配置持久化
  ├── utils/cwd.ts       → 获取当前目录
  ├── utils/file.ts      → 目录空检查
  └── screens/REPL.tsx   → 每次提交时检查引导状态
```

## 总结

一个简洁的状态管理模块，用条件步骤系统引导用户完成项目初始设置。设计上注重性能（缓存短路）和可扩展性（步骤列表可轻松添加新步骤）。
