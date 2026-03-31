# Claude Code 背景知识

> 本文面向零基础读者，帮助你理解 Claude Code 项目的技术领域和核心概念。

---

## 项目简介

**Claude Code** 是 Anthropic 公司开发的一款**终端 AI 编程助手 CLI 工具**。用户可以通过终端与 Claude 大语言模型对话，让它帮助完成各种软件工程任务：编辑文件、运行命令、搜索代码库、协调工作流程等。

**关键特点**：
- 📦 **规模**：~1,884 个 TypeScript 文件，512,000+ 行代码
- 🏗️ **运行时**：Bun（高性能 JavaScript/TypeScript 运行时）
- 🖥️ **终端 UI**：React + [Ink](https://github.com/vadimdemedes/ink)（将 React 组件渲染到终端）
- 🤖 **AI 集成**：通过 Anthropic API 与 Claude 模型交互
- 🔌 **扩展性**：支持 MCP 服务器、插件系统、Skill 系统

---

## 核心概念

### 1. 终端 UI (TUI) — Ink 框架

与传统的 Web 界面不同，Claude Code 使用 **Ink** 将 React 组件渲染到终端。就像 React Native 将 React 渲染到手机屏幕一样，Ink 将 React 渲染到终端窗口。

```
┌──────────────────────────────────────┐
│  Terminal (iTerm2, Terminal.app, etc.) │
│  ┌──────────────────────────────────┐ │
│  │     Ink React Components         │ │
│  │  ┌────────────────────────────┐  │ │
│  │  │  REPL Screen               │  │ │
│  │  │  ├── Message List          │  │ │
│  │  │  ├── Tool Progress         │  │ │
│  │  │  └── Prompt Input          │  │ │
│  │  └────────────────────────────┘  │ │
│  └──────────────────────────────────┘ │
└──────────────────────────────────────┘
```

### 2. Tool 系统 — AI 的"手和脚"

Claude 模型本身只能生成文本，但通过 **Tool（工具）系统**，它可以执行实际操作：

| 工具 | 功能 |
|------|------|
| `BashTool` | 在 shell 中执行命令 |
| `FileReadTool` | 读取文件内容 |
| `FileEditTool` | 编辑文件（查找-替换） |
| `FileWriteTool` | 写入新文件 |
| `GlobTool` | 按模式搜索文件名 |
| `GrepTool` | 搜索文件内容 |
| `WebFetchTool` | 发起 HTTP 请求 |
| `WebSearchTool` | 网页搜索 |
| `AgentTool` | 创建子代理（多代理协作） |
| `SkillTool` | 调用技能/命令 |
| `TodoWriteTool` | 任务管理 |

### 3. 查询循环 — 对话的心跳

Claude Code 的核心是一个**查询循环（Query Loop）**：

```
用户输入 → 构建系统提示 → 调用 Claude API → 解析响应
    ↑                                           |
    |     ← 文本回复 ← end_turn                  |
    |                                           ↓
    └──── 工具结果 ← Tool.call() ← tool_use 指令
```

每一轮对话中，Claude 可能：
1. 直接返回文本（对话结束）
2. 调用一个或多个工具（工具执行后结果反馈给 Claude，继续循环）

### 4. 权限系统 — 安全的守门人

Claude Code 拥有访问文件系统和执行命令的能力，因此需要严格的权限控制：

- **Permission Mode**：`default`（每次询问）、`auto`（自动判断安全性）、`bypassPermissions`（沙盒模式全部跳过）
- **工具级权限**：每个工具有 `checkPermissions()` 方法，在执行前检查
- **钩子系统**：`PreToolUse` / `PostToolUse` 钩子允许自定义权限逻辑
- **Auto Mode 分类器**：使用 AI 判断工具调用是否安全

### 5. Feature Flag 系统 — 条件编译

Claude Code 大量使用 **Feature Flag** 来控制功能开关：

```typescript
import { feature } from 'bun:bundle'

// 在构建时进行死代码消除
const coordinatorModule = feature('COORDINATOR_MODE')
  ? require('./coordinator/coordinatorMode.js')
  : null
```

这意味着不同构建版本可以包含不同的功能集。常见 Feature Flag 包括：
- `COORDINATOR_MODE` — 多代理协调器模式
- `KAIROS` — 助手模式（Kairos）
- `BRIDGE_MODE` — IDE 桥接模式
- `VOICE_MODE` — 语音输入
- `WORKFLOW_SCRIPTS` — 工作流脚本

### 6. MCP (Model Context Protocol) — 扩展协议

MCP 是一种开放协议，允许外部服务为 AI 模型提供工具和上下文：

```
Claude Code ←→ MCP Server A (GitHub)
            ←→ MCP Server B (数据库)
            ←→ MCP Server C (自定义)
```

### 7. 多代理系统 — Coordinator 模式

Claude Code 支持**多代理协作**，一个 Coordinator（协调者）可以创建多个 Worker（工作者）并行执行任务：

```
            ┌─── Worker A (研究) ───┐
用户 → Coordinator ─── Worker B (实现) ───→ 综合结果
            └─── Worker C (验证) ───┘
```

---

## 技术栈总结

| 层 | 技术 |
|---|------|
| 语言 | TypeScript |
| 运行时 | Bun |
| 终端 UI | React + Ink |
| CLI 框架 | Commander.js |
| API 客户端 | @anthropic-ai/sdk |
| Schema 验证 | Zod v4 |
| 状态管理 | 自建轻量级 Store |
| 构建系统 | Bun bundler (feature flags) |
| 包管理 | npm |

---

## 架构总览图

```
┌──────────────────────────────────────────────────────┐
│                    main.tsx (入口)                     │
│          Commander.js CLI 解析 + 初始化                │
├──────────────┬───────────────────────────────────────┤
│   交互模式    │           非交互模式 (-p)               │
│  launchRepl() │         runHeadless()                 │
│    ↓          │              ↓                        │
│  <App>        │        QueryEngine                    │
│   <REPL />    │              ↓                        │
│  </App>       │          query()                      │
├──────────────┴───────────────────────────────────────┤
│                  query() 查询循环                      │
│     processUserInput → claude.ts API → Tool.call()   │
├──────────────────────────────────────────────────────┤
│                    核心子系统                           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐│
│  │ commands/ │ │  tools/  │ │services/ │ │  hooks/  ││
│  │ (50+命令) │ │(40+工具) │ │(API/MCP/ │ │(权限/    ││
│  │          │ │          │ │ LSP/...)  │ │ 通知/...)││
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘│
├──────────────────────────────────────────────────────┤
│                    基础设施                            │
│  utils/ | state/ | types/ | constants/ | skills/     │
│  plugins/ | bridge/ | coordinator/ | ink/             │
└──────────────────────────────────────────────────────┘
```
