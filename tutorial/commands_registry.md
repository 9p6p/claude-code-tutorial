# `commands.ts` — 斜杠命令注册表

## 文件概述

`commands.ts` 是 Claude Code 所有**斜杠命令**（Slash Commands）的注册中心。用户在 REPL 中输入 `/help`、`/compact`、`/config` 等命令时，就是这个文件负责找到并执行对应的处理逻辑。它大约有 300+ 行代码，核心职责是维护一个命令数组并提供查询接口。

## 关键代码解读

### 1. 命令导入区（60+ 个命令）

```typescript
import addDir from './commands/add-dir/index.js'
import clear from './commands/clear/index.js'
import compact from './commands/compact/index.js'
import config from './commands/config/index.js'
import cost from './commands/cost/index.js'
import diff from './commands/diff/index.js'
import doctor from './commands/doctor/index.js'
import help from './commands/help/index.js'
// ... 60+ 个命令导入
```

每个命令独立存放在 `commands/` 目录下，以模块化方式组织。

### 2. Feature Flag 条件命令

```typescript
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null

const bridge = feature('BRIDGE_MODE')
  ? require('./commands/bridge/index.js').default
  : null

const buddy = feature('BUDDY')
  ? require('./commands/buddy/index.js').default
  : null
```

实验性或平台特定的命令通过 `bun:bundle` 的 feature flag 按需加载，确保未启用的功能不会增加打包体积。

### 3. 命令注册列表

```typescript
const COMMANDS = memoize((): Command[] => [
  addDir, advisor, agents, branch, clear,
  color, compact, config, copy, cost,
  diff, doctor, effort, exit, fast,
  files, help, ide, init, keybindings,
  mcp, memory, mobile, model, outputStyle,
  plugin, pr_comments, releaseNotes, rename,
  resume, review, securityReview, session,
  share, skills, status, tasks, theme,
  vim, usage, ...
])
```

使用 `lodash.memoize` 缓存命令列表，避免重复创建。

### 4. 内部专用命令

```typescript
export const INTERNAL_ONLY_COMMANDS = [
  backfillSessions, breakCache, bughunter,
  commit, commitPushPr, ctx_viz, goodClaude,
  issue, initVerifiers, mockLimits, ...
]
```

这些命令仅供 Anthropic 内部使用（`ant` 用户类型），在外部构建中被排除。

### 5. 获取命令的核心函数

```typescript
export function getCommands(): Command[] {
  return [
    ...COMMANDS(),
    ...getSkillDirCommands(),       // 从 skills 目录加载
    ...getBundledSkills(),           // 内置 skills
    ...getBuiltinPluginSkillCommands(), // 插件 skills
    ...getPluginCommands(),          // 插件命令
    ...getPluginSkills(),            // 插件 skills
  ]
}
```

最终的命令列表不仅包含内置命令，还动态合并了 Skills 和 Plugins 提供的命令。

## 核心类型定义

命令类型在 `types/command.ts` 中定义：

```typescript
type Command = {
  type: 'prompt' | 'local' | 'local-jsx'
  name: string
  description: string
  source: 'builtin' | 'skill' | 'plugin'
  isEnabled?: () => boolean
  // ... 更多属性
}
```

| 命令类型 | 说明 |
|---------|------|
| `prompt` | 生成提示词发送给 LLM（如 `/review`） |
| `local` | 本地执行，不涉及 LLM（如 `/clear`） |
| `local-jsx` | 本地执行并渲染 JSX UI（如 `/config`） |

## 与其他模块的关系

```
commands.ts
  ├── commands/*/index.ts → 各个命令的实现
  ├── skills/loadSkillsDir.ts → 加载用户自定义 Skills
  ├── skills/bundledSkills.ts → 内置 Skills
  ├── plugins/builtinPlugins.ts → 内置插件命令
  ├── utils/plugins/loadPluginCommands.ts → 外部插件命令
  └── types/command.ts → Command 类型定义
```

## 总结

`commands.ts` 是一个典型的**注册表模式**（Registry Pattern）实现。它的设计亮点：
- 静态命令 + 动态扩展（Skills/Plugins）的混合加载
- Feature Flag 控制实验性命令的可用性
- Memoize 缓存避免重复创建
- 清晰的命令类型分类（prompt/local/local-jsx）
