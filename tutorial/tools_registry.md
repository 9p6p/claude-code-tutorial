# `tools.ts` — AI 工具注册表

## 文件概述

`tools.ts` 是 Claude Code 所有 **AI 工具**（Tools）的注册中心。当 Claude AI 模型决定调用工具（如读取文件、执行 Bash 命令、搜索代码）时，就是这个文件提供的工具列表告诉模型有哪些工具可用。大约 300 行代码，职责是管理工具的注册、过滤和权限检查。

## 关键代码解读

### 1. 核心工具导入

```typescript
import { AgentTool } from './tools/AgentTool/AgentTool.js'
import { BashTool } from './tools/BashTool/BashTool.js'
import { FileEditTool } from './tools/FileEditTool/FileEditTool.js'
import { FileReadTool } from './tools/FileReadTool/FileReadTool.js'
import { FileWriteTool } from './tools/FileWriteTool/FileWriteTool.js'
import { GlobTool } from './tools/GlobTool/GlobTool.js'
import { GrepTool } from './tools/GrepTool/GrepTool.js'
import { WebFetchTool } from './tools/WebFetchTool/WebFetchTool.js'
import { WebSearchTool } from './tools/WebSearchTool/WebSearchTool.js'
import { TodoWriteTool } from './tools/TodoWriteTool/TodoWriteTool.js'
```

这些是 Claude Code 的**核心工具**，几乎在所有会话中都可用。

### 2. getAllBaseTools() — 工具注册表

```typescript
export function getAllBaseTools(): Tools {
  return [
    AgentTool,          // 创建子 Agent
    TaskOutputTool,     // 查看任务输出
    BashTool,           // 执行 Bash 命令
    GlobTool,           // 文件模式匹配搜索
    GrepTool,           // 内容搜索
    FileReadTool,       // 读取文件
    FileEditTool,       // 编辑文件
    FileWriteTool,      // 写入文件
    WebFetchTool,       // 抓取网页
    TodoWriteTool,      // Todo 管理
    WebSearchTool,      // 网络搜索
    SkillTool,          // 运行 Skills
    AskUserQuestionTool, // 向用户提问
    EnterPlanModeTool,  // 进入规划模式
    // ... 条件性加载的工具
  ]
}
```

这是所有可能可用工具的**完整列表**，是系统的单一事实来源（Single Source of Truth）。

### 3. Feature Flag 条件工具

```typescript
const SleepTool = feature('PROACTIVE') || feature('KAIROS')
  ? require('./tools/SleepTool/SleepTool.js').SleepTool
  : null

const WorkflowTool = feature('WORKFLOW_SCRIPTS')
  ? (() => {
      require('./tools/WorkflowTool/bundled/index.js').initBundledWorkflows()
      return require('./tools/WorkflowTool/WorkflowTool.js').WorkflowTool
    })()
  : null
```

与命令系统类似，实验性工具通过 Feature Flag 控制。

### 4. 权限过滤 — getTools()

```typescript
export const getTools = (permissionContext: ToolPermissionContext): Tools => {
  // 简单模式：只有 Bash、Read、Edit
  if (isEnvTruthy(process.env.CLAUDE_CODE_SIMPLE)) {
    return filterToolsByDenyRules(
      [BashTool, FileReadTool, FileEditTool],
      permissionContext
    )
  }
  
  // 完整模式：所有工具 - 被拒绝的工具
  return filterToolsByDenyRules(getAllBaseTools(), permissionContext)
}
```

根据权限上下文过滤掉被全局拒绝的工具，确保模型看不到不允许使用的工具。

### 5. 权限拒绝规则过滤

```typescript
export function filterToolsByDenyRules<T>(
  tools: readonly T[],
  permissionContext: ToolPermissionContext
): T[] {
  return tools.filter(
    tool => !getDenyRuleForTool(permissionContext, tool)
  )
}
```

这个函数实现了**工具级权限控制**——如果权限配置中有对某个工具的全面拒绝规则，该工具会在模型看到工具列表之前就被移除。

## 工具预设系统

```typescript
export const TOOL_PRESETS = ['default'] as const

export function getToolsForDefaultPreset(): string[] {
  const tools = getAllBaseTools()
  const isEnabled = tools.map(tool => tool.isEnabled())
  return tools.filter((_, i) => isEnabled[i]).map(tool => tool.name)
}
```

支持工具预设概念，目前只有 `default` 一种预设。

## 与其他模块的关系

```
tools.ts
  ├── tools/*/Tool.ts       → 各个工具的实现
  ├── Tool.ts               → Tool 类型定义 & ToolUseContext
  ├── constants/tools.ts    → 工具常量（禁用列表等）
  ├── utils/permissions/    → 权限过滤逻辑
  └── utils/envUtils.ts     → 环境变量检查
```

## 总结

`tools.ts` 与 `commands.ts` 构成了 Claude Code 的"双注册表"：
- `commands.ts`：用户直接使用的斜杠命令
- `tools.ts`：AI 模型使用的工具

设计亮点：
- **单一事实来源**：`getAllBaseTools()` 包含所有工具
- **编译时裁剪**：Feature Flag 移除未启用工具
- **运行时权限**：`filterToolsByDenyRules` 按权限过滤
- **环境感知**：嵌入式搜索工具检测（避免重复功能）
