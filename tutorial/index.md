# Claude Code 源码教程

[![GitHub](https://img.shields.io/badge/GitHub-教程源码仓库-181717?logo=github&logoColor=white)](https://github.com/9p6p/claude-code-tutorial)
[![Claude Code](https://img.shields.io/badge/Claude_Code-原始项目-7C3AED?logo=anthropic&logoColor=white)](https://github.com/anthropics/claude-code)
[![Pages](https://img.shields.io/badge/在线阅读-GitHub_Pages-2088FF?logo=github-pages&logoColor=white)](https://9p6p.github.io/claude-code-tutorial/)

**Claude Code** 是 Anthropic 开发的终端 AI 编程助手，这是一份面向开发者的逐文件源码解读教程。

[:fontawesome-brands-github: **本教程源码仓库**](https://github.com/9p6p/claude-code-tutorial){ .md-button .md-button--primary }
[:material-book-open-variant: **Claude Code 原始项目**](https://github.com/anthropics/claude-code){ .md-button }

---

## 🏗️ 架构总览

```
                          ┌─────────────────┐
                          │   用户终端输入    │
                          └────────┬────────┘
                                   │
                          ┌────────▼────────┐
                          │   main.tsx       │
                          │ Commander.js CLI │
                          └───┬─────────┬───┘
                              │         │
                   交互模式    │         │   非交互模式 (-p)
                              │         │
                    ┌─────────▼──┐  ┌───▼──────────┐
                    │ launchRepl │  │ runHeadless   │
                    │  <App>     │  │ (print.ts)    │
                    │   <REPL /> │  └───┬───────────┘
                    └─────┬──────┘      │
                          │             │
                    ┌─────▼─────────────▼─────┐
                    │     QueryEngine          │
                    │  submitMessage() 循环     │
                    └─────────┬───────────────┘
                              │
                    ┌─────────▼───────────────┐
                    │      query() 核心循环     │
                    │  ┌───────────────────┐   │
                    │  │ claude.ts API 调用 │   │
                    │  └───────┬───────────┘   │
                    │          │               │
                    │     ┌────▼────┐          │
                    │     │ 解析响应 │          │
                    │     └────┬────┘          │
                    │          │               │
                    │  ┌───────▼───────────┐   │
                    │  │ text → 返回用户    │   │
                    │  │ tool_use → 执行    │   │
                    │  │   Tool.call()      │   │
                    │  │   结果 → 继续循环   │   │
                    │  └───────────────────┘   │
                    └─────────────────────────┘
```

## 📊 主工作流

```
Step 1: CLI 启动
  main.tsx → 解析命令行参数 (Commander.js)
  → init() 初始化 → setup() 配置环境
       │
Step 2: 分支
  ├─ 交互模式 → launchRepl() → <App><REPL /></App>
  │    └→ 等待用户在终端输入
  └─ 非交互模式 (-p) → runHeadless() via print.ts
       └→ 直接处理管道输入
       │
Step 3: 处理用户输入
  processUserInput() → 解析 /slash 命令
  → 注入 CLAUDE.md 记忆 → 构建消息
       │
Step 4: 查询 LLM
  QueryEngine.submitMessage()
  → fetchSystemPromptParts() 构建系统提示
  → query() → claude.ts 流式 API 调用
       │
Step 5: 工具执行
  解析 tool_use 块 → 权限检查
  → Tool.call() 执行 → 结果注入消息
  → 继续查询循环直到 end_turn
       │
Step 6: 结果返回
  文本结果 → 渲染到终端 (Ink)
  费用追踪 → cost-tracker.ts
  会话持久化 → sessionStorage.ts
```

## 📂 代码→工作流映射表

| 工作流步骤 | 源文件 | 教程文档 |
|-----------|--------|---------|
| CLI 入口 & 参数解析 | `src/main.tsx` | [main.md](./main.md) |
| 命令注册 | `src/commands.ts` | [commands_registry.md](./commands_registry.md) |
| 工具注册 | `src/tools.ts` | [tools_registry.md](./tools_registry.md) |
| 工具类型系统 | `src/Tool.ts` | [Tool.md](./Tool.md) |
| 查询引擎 | `src/QueryEngine.ts` | [QueryEngine.md](./QueryEngine.md) |
| 核心查询循环 | `src/query.ts` | [query.md](./query.md) |
| API 交互层 | `src/services/api/claude.ts` | [services/index.md](./services/index.md) |
| 上下文收集 | `src/context.ts` | [context.md](./context.md) |
| 初始化 | `src/setup.ts` | [setup.md](./setup.md) |
| 费用追踪 | `src/cost-tracker.ts` | [cost-tracker.md](./cost-tracker.md) |
| REPL 启动 | `src/replLauncher.tsx` | [replLauncher.md](./replLauncher.md) |
| 多代理协调 | `src/coordinator/coordinatorMode.ts` | [coordinator/index.md](./coordinator/index.md) |

## 📦 模块索引

| 模块 | 文件数 | 描述 |
|------|--------|------|
| [核心入口](./00_reading_guide.md#核心入口src-根目录) | 18 | CLI 入口、注册中心、查询引擎 |
| [commands/](./commands/index.md) | 189 | 50+ 斜杠命令 |
| [tools/](./tools/index.md) | 184 | 40+ 代理工具 |
| [components/](./components/index.md) | 389 | Ink UI 组件 |
| [services/](./services/index.md) | 130 | 外部服务集成 |
| [utils/](./utils/index.md) | 564 | 工具函数库 |
| [hooks/](./hooks/index.md) | 104 | React Hooks |
| [ink/](./ink/index.md) | 96 | 终端渲染引擎 |
| [bridge/](./bridge/index.md) | 31 | IDE 桥接 |
| [skills/](./skills/index.md) | 20 | Skill 系统 |
| [tasks/](./tasks/index.md) | 12 | 任务系统 |
| [state/](./state/index.md) | 6 | 状态管理 |
| [coordinator/](./coordinator/index.md) | 1 | 多代理协调 |

## 🚀 快速开始

- **零基础？** 先阅读 [背景知识](./00_background_knowledge.md)
- **想快速了解？** 按 [快速入门路径](./00_reading_guide.md#-快速入门路径2-小时) 阅读
- **想深入研究？** 参考 [完整学习路径](./00_reading_guide.md#-完整学习路径2-3-天)
- **只关心某个主题？** 查看 [专题阅读路径](./00_reading_guide.md#-专题阅读路径)

---

> **生成时间**：2026-03-31
> **项目版本**：Claude Code 源码快照（通过 npm source map 公开暴露）
> **教程语言**：中文
