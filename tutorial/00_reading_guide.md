# Claude Code 源码阅读指南

> 本指南帮助你规划阅读路径，高效理解这个 512,000+ 行的大型 TypeScript 项目。

---

## 📖 三种阅读路径

### 🚀 快速入门路径（~2 小时）

适合：想快速理解项目运作方式的读者。

| 步骤 | 文件 | 重点 |
|------|------|------|
| 1 | `src/main.tsx` (前 200 行) | CLI 入口、启动流程概览 |
| 2 | `src/commands.ts` | 命令注册机制、优先级层次 |
| 3 | `src/tools.ts` | 工具注册机制、feature flag 模式 |
| 4 | `src/Tool.ts` (前 300 行) | 工具类型定义、`buildTool()` 工厂 |
| 5 | `src/QueryEngine.ts` (前 200 行) | 查询引擎核心、`submitMessage()` 流程 |
| 6 | `src/context.ts` | 上下文收集（git状态、CLAUDE.md） |
| 7 | `src/replLauncher.tsx` | REPL 启动（仅 23 行，极简） |
| 8 | 任意一个 Tool 实现（如 `src/tools/FileEditTool/`）| 工具的完整生命周期 |

### 📚 完整学习路径（~2-3 天）

适合：需要深入理解架构的开发者。

**第一阶段：核心骨架**（~4 小时）
1. 入口与启动：`main.tsx` → `setup.ts` → `replLauncher.tsx`
2. 命令系统：`commands.ts` → `types/command.ts` → 选读几个 `commands/` 实现
3. 工具系统：`Tool.ts` → `tools.ts` → 选读 `BashTool` 和 `FileEditTool`
4. 查询循环：`QueryEngine.ts` → `query.ts` → `services/api/claude.ts`

**第二阶段：UI 与交互**（~3 小时）
5. 终端渲染：`ink/` 目录（重点：`ink.tsx`、`render-node-to-output.ts`、`components/`）
6. REPL 界面：`screens/REPL.tsx`（874KB 超大文件，选读核心逻辑）
7. 组件库：`components/` 目录（重点：`PromptInput/`、`messages/`、`permissions/`）

**第三阶段：扩展系统**（~3 小时）
8. 权限系统：`utils/permissions/` → `hooks/useCanUseTool.ts`
9. 插件系统：`plugins/` → `utils/plugins/`
10. Skill 系统：`skills/` → `tools/SkillTool/`
11. MCP 集成：`services/mcp/`

**第四阶段：高级功能**（~3 小时）
12. 多代理：`coordinator/coordinatorMode.ts` → `tools/AgentTool/` → `tasks/`
13. 桥接模式：`bridge/`
14. 状态管理：`state/` → `bootstrap/state.ts`
15. API 层：`services/api/claude.ts`（122KB，核心中的核心）

### 🎯 专题阅读路径

按兴趣选择模块深入：

| 主题 | 核心文件 |
|------|---------|
| **工具开发** | `Tool.ts` → 任意 `tools/XxxTool/` → `utils/permissions/` |
| **命令开发** | `types/command.ts` → `commands.ts` → 任意 `commands/xxx/` |
| **终端 UI** | `ink/` 全部 → `components/` → `screens/REPL.tsx` |
| **API 交互** | `services/api/claude.ts` → `services/api/withRetry.ts` → `services/api/errors.ts` |
| **权限安全** | `utils/permissions/` → `hooks/useCanUseTool.ts` → `utils/hooks/` |
| **多代理** | `coordinator/` → `tools/AgentTool/` → `tasks/` → `utils/swarm/` |
| **插件系统** | `plugins/` → `utils/plugins/` → `commands/plugin/` |
| **MCP 集成** | `services/mcp/` → `tools/MCPTool/` → `tools/ListMcpResourcesTool/` |
| **会话持久化** | `utils/sessionStorage.ts` → `utils/conversationRecovery.ts` → `history.ts` |
| **成本追踪** | `cost-tracker.ts` → `bootstrap/state.ts` → `utils/modelCost.ts` |

---

## 📁 模块索引

### 核心入口（src/ 根目录）

| 文件 | 描述 | 重要性 |
|------|------|--------|
| `main.tsx` | CLI 入口点（4,684 行）| ⭐⭐⭐⭐⭐ |
| `commands.ts` | 命令注册中心 | ⭐⭐⭐⭐⭐ |
| `tools.ts` | 工具注册中心 | ⭐⭐⭐⭐⭐ |
| `Tool.ts` | 工具类型定义（793 行）| ⭐⭐⭐⭐⭐ |
| `QueryEngine.ts` | LLM 查询引擎（1,296 行）| ⭐⭐⭐⭐⭐ |
| `query.ts` | 核心查询循环（1,730 行）| ⭐⭐⭐⭐ |
| `context.ts` | 系统/用户上下文收集 | ⭐⭐⭐⭐ |
| `setup.ts` | 初始化设置（478 行）| ⭐⭐⭐ |
| `cost-tracker.ts` | Token 费用追踪 | ⭐⭐⭐ |
| `Task.ts` | 任务类型定义 | ⭐⭐⭐ |
| `replLauncher.tsx` | REPL 启动器（23 行）| ⭐⭐ |

### 模块目录

| 目录 | 文件数 | 描述 |
|------|--------|------|
| `utils/` | 564 | 工具函数（最大模块：bash、permissions、model、settings、plugins） |
| `components/` | 389 | Ink UI 组件（PromptInput、messages、permissions、Settings 等） |
| `commands/` | 189 | 50+ 斜杠命令实现 |
| `tools/` | 184 | 40+ 代理工具实现 |
| `services/` | 130 | 外部服务集成（API、MCP、LSP、OAuth、analytics） |
| `hooks/` | 104 | React Hooks（通知、工具权限、状态管理） |
| `ink/` | 96 | 终端 UI 渲染引擎（Ink fork/增强版） |
| `bridge/` | 31 | IDE 与远程控制桥接 |
| `constants/` | 21 | 全局常量定义 |
| `skills/` | 20 | Skill 系统（bundled skills + skill loader） |
| `cli/` | 19 | CLI 传输层和处理器 |
| `keybindings/` | 14 | 键盘快捷键配置 |
| `tasks/` | 12 | 任务系统（DreamTask、LocalAgent、RemoteAgent、Shell） |
| `types/` | 11 | 全局 TypeScript 类型定义 |
| `migrations/` | 11 | 配置迁移脚本 |
| `context/` | 9 | 上下文管理（stats、notifications） |
| `entrypoints/` | 8 | 入口点配置（CLI、SDK、MCP） |
| `memdir/` | 8 | 记忆系统（CLAUDE.md 管理） |
| `buddy/` | 6 | Buddy 功能模块 |
| `state/` | 6 | 状态管理（AppState、Store、onChange） |
| `vim/` | 5 | Vim 模式实现 |
| `query/` | 4 | 查询相关模块 |
| `remote/` | 4 | 远程会话管理 |
| `native-ts/` | 4 | 原生 TS 实现（color-diff、file-index、yoga-layout） |
| `server/` | 3 | 服务器端（Direct Connect） |
| `screens/` | 3 | 全屏 UI（REPL、Doctor、Resume） |
| `plugins/` | 2 | 插件系统核心 |
| `upstreamproxy/` | 2 | 上游代理 |
| `coordinator/` | 1 | 多代理协调器 |
| `voice/` | 1 | 语音输入 |

---

## 🔑 阅读技巧

### 1. Feature Flag 模式
遇到 `feature('XXX')` 时，可以暂时忽略条件分支中的代码。这些是可选功能，不影响核心逻辑理解。

### 2. 超大文件策略
- `main.tsx`（4,684 行）：只需读 `main()` 和 `run()` 函数的骨架
- `REPL.tsx`（874KB）：关注 state 定义和核心 useEffect
- `claude.ts`（122KB）：关注 `makeApiRequest` 和消息规范化

### 3. 构建时常量
`"external"` 和 `process.env.USER_TYPE` 在构建时确定，`'ant'` 表示内部版本。公开版本中这些分支被消除。

### 4. 延迟导入模式
项目大量使用 `require()` 包裹在条件中，结合 `feature()` 实现构建时的死代码消除。这是性能优化手段。

---

## 🔗 快速跳转

- [背景知识](./00_background_knowledge.md) — 核心概念入门
- [进度跟踪](./TODO.md) — 文档完成度
