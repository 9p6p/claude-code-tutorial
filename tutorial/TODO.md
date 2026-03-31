# Claude Code 源码教程 - 进度跟踪

> **总计**: ~1,884 个源码文件待文档化
> **创建时间**: 2026-03-31
> **目标语言**: 中文 (Chinese)
> **项目**: Claude Code CLI — Anthropic 的终端 AI 编程助手

## 进度概览

| 模块 | 文件数 | 已完成 | 状态 |
|------|--------|--------|------|
| src/ (根文件) | 18 | 0 | ⏳ 待处理 |
| assistant | 1 | 0 | ⏳ 待处理 |
| bootstrap | 1 | 0 | ⏳ 待处理 |
| bridge | 31 | 0 | ⏳ 待处理 |
| buddy | 6 | 0 | ⏳ 待处理 |
| cli | 19 | 0 | ⏳ 待处理 |
| commands | 189 | 0 | ⏳ 待处理 |
| components | 389 | 0 | ⏳ 待处理 |
| constants | 21 | 0 | ⏳ 待处理 |
| context | 9 | 0 | ⏳ 待处理 |
| coordinator | 1 | 0 | ⏳ 待处理 |
| entrypoints | 8 | 0 | ⏳ 待处理 |
| hooks | 104 | 0 | ⏳ 待处理 |
| ink | 96 | 0 | ⏳ 待处理 |
| keybindings | 14 | 0 | ⏳ 待处理 |
| memdir | 8 | 0 | ⏳ 待处理 |
| migrations | 11 | 0 | ⏳ 待处理 |
| moreright | 1 | 0 | ⏳ 待处理 |
| native-ts | 4 | 0 | ⏳ 待处理 |
| outputStyles | 1 | 0 | ⏳ 待处理 |
| plugins | 2 | 0 | ⏳ 待处理 |
| query | 4 | 0 | ⏳ 待处理 |
| remote | 4 | 0 | ⏳ 待处理 |
| schemas | 1 | 0 | ⏳ 待处理 |
| screens | 3 | 0 | ⏳ 待处理 |
| server | 3 | 0 | ⏳ 待处理 |
| services | 130 | 0 | ⏳ 待处理 |
| skills | 20 | 0 | ⏳ 待处理 |
| state | 6 | 0 | ⏳ 待处理 |
| tasks | 12 | 0 | ⏳ 待处理 |
| tools | 184 | 0 | ⏳ 待处理 |
| types | 11 | 0 | ⏳ 待处理 |
| upstreamproxy | 2 | 0 | ⏳ 待处理 |
| utils | 564 | 0 | ⏳ 待处理 |
| vim | 5 | 0 | ⏳ 待处理 |
| voice | 1 | 0 | ⏳ 待处理 |

## 核心入口文件 (src/)

- [ ] `src/main.tsx` → `tutorial/main.md` — 应用入口点
- [ ] `src/commands.ts` → `tutorial/commands_registry.md` — 命令注册中心
- [ ] `src/tools.ts` → `tutorial/tools_registry.md` — 工具注册中心
- [ ] `src/Tool.ts` → `tutorial/Tool.md` — 工具类型定义
- [ ] `src/QueryEngine.ts` → `tutorial/QueryEngine.md` — LLM 查询引擎
- [ ] `src/context.ts` → `tutorial/context.md` — 系统/用户上下文收集
- [ ] `src/cost-tracker.ts` → `tutorial/cost-tracker.md` — Token 费用追踪
- [ ] `src/Task.ts` → `tutorial/Task.md` — 任务类型定义
- [ ] `src/history.ts` → `tutorial/history.md` — 会话历史
- [ ] `src/ink.ts` → `tutorial/ink_entry.md` — Ink 渲染入口
- [ ] `src/setup.ts` → `tutorial/setup.md` — 初始化设置
- [ ] `src/query.ts` → `tutorial/query.md` — 查询接口
- [ ] `src/tasks.ts` → `tutorial/tasks_entry.md` — 任务系统入口
- [ ] `src/costHook.ts` → `tutorial/costHook.md` — 费用 Hook
- [ ] `src/dialogLaunchers.tsx` → `tutorial/dialogLaunchers.md` — 对话框启动器
- [ ] `src/interactiveHelpers.tsx` → `tutorial/interactiveHelpers.md` — 交互辅助
- [ ] `src/replLauncher.tsx` → `tutorial/replLauncher.md` — REPL 启动器
- [ ] `src/projectOnboardingState.ts` → `tutorial/projectOnboardingState.md` — 项目引导状态

## 模块文件清单

### assistant/
- [ ] `src/assistant/sessionHistory.ts` → `tutorial/assistant/sessionHistory.md`

### bootstrap/
- [ ] `src/bootstrap/bootstrap.ts` → `tutorial/bootstrap/bootstrap.md`

### bridge/ (31 files)
- [ ] IDE 与远程控制桥接模块（详细文件列表待展开）

### tools/ (184 files)
- [ ] 40+ 工具实现（BashTool, FileEditTool, FileReadTool, GlobTool, GrepTool 等）

### commands/ (189 files)
- [ ] 50+ 斜杠命令实现（compact, config, doctor, help, login, mcp, memory 等）

### components/ (389 files)
- [ ] 140+ Ink UI 组件（PromptInput, Settings, permissions, messages 等）

### services/ (130 files)
- [ ] 外部服务集成（API, MCP, LSP, OAuth, analytics 等）

### utils/ (564 files)
- [ ] 工具函数（bash, git, permissions, model, settings, plugins 等）

### hooks/ (104 files)
- [ ] React Hooks（通知、工具权限、状态管理等）

### ink/ (96 files)
- [ ] 终端 UI 渲染引擎（组件、事件、布局、hooks 等）

---

> **注意**: 由于项目规模庞大 (512,000+ 行代码)，将采用分阶段并行方式逐模块生成文档。
