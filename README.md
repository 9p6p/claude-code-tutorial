# Claude Code 源码教程

[![GitHub Pages](https://img.shields.io/badge/在线阅读-GitHub_Pages-2088FF?logo=github-pages&logoColor=white)](https://9p6p.github.io/claude-code-tutorial/)
[![GitHub Repo](https://img.shields.io/badge/GitHub-教程源码仓库-181717?logo=github&logoColor=white)](https://github.com/9p6p/claude-code-tutorial)
[![Claude Code](https://img.shields.io/badge/Claude_Code-原始项目-7C3AED?logo=anthropic&logoColor=white)](https://github.com/anthropics/claude-code)
[![Docs](https://img.shields.io/badge/文档-53份_·_4400行-green)](#-文档覆盖)

> 📖 对 [Claude Code](https://github.com/anthropics/claude-code) 项目的深度逐文件代码教程，覆盖**全部 36 个模块**、512,000+ 行 TypeScript 源码。

## 🌐 在线阅读

👉 **[https://9p6p.github.io/claude-code-tutorial/](https://9p6p.github.io/claude-code-tutorial/)**

## 🔗 相关链接

| 链接 | 说明 |
|------|------|
| 📘 [**教程在线阅读**](https://9p6p.github.io/claude-code-tutorial/) | MkDocs Material 主题，支持亮暗模式 |
| 💻 [**教程源码仓库**](https://github.com/9p6p/claude-code-tutorial) | 本教程的 Markdown 源文件 |
| 🏠 [**Claude Code 原始项目**](https://github.com/anthropics/claude-code) | Anthropic 官方仓库 |

## ✨ 特性

- 📚 **53 份教程文档**，全中文，合计 **4,400 行 / 268KB**
- 🧰 **33+ AI 工具**深度解析（BashTool、FileEditTool、AgentTool 等）
- ⌨️ **67+ 斜杠命令**完整索引与分类
- 🧩 **439 个 UI 组件**架构分析
- 🪝 **104 个 React Hooks** 逐一分析
- 🔧 **564 个工具函数** 8 大子系统深度拆解
- 🎨 **Material 主题**，支持亮色/暗色模式切换
- 🔍 **中文全文搜索**
- 📊 **架构图和流程图**（Mermaid 渲染）
- 💻 **代码高亮** + 一键复制

## 📁 文档覆盖

### 核心入口文件（18 份）

`main.tsx` · `commands.ts` · `tools.ts` · `Tool.ts` · `QueryEngine.ts` · `query.ts` · `context.ts` · `setup.ts` · `ink.ts` · `Task.ts` · `tasks.ts` · `history.ts` · `cost-tracker.ts` · `costHook.ts` · `replLauncher.tsx` · `dialogLaunchers.tsx` · `interactiveHelpers.tsx` · `projectOnboardingState.ts`

### 核心实现模块（7 份）

| 模块 | 源码文件数 | 文档亮点 |
|------|-----------|---------|
| ⚙️ **tools/** | 184 | 33+ 工具完整索引，6 大分类，安全层深度分析 |
| ⌨️ **commands/** | 189 | 67+ 命令分类表，懒加载架构，三种命令类型解析 |
| 🧩 **components/** | 389 | 439 文件分类索引，超级组件 Top 5，React Compiler 实践 |
| 🪝 **hooks/** | 104 | 15 个核心 Hook 逐一分析，状态/效果/返回值/用法 |
| 🔧 **utils/** | 564 | 8 大子系统深度拆解，权限 6 模式 / 配置 7 来源 / YOLO 分类器 |
| 🖥️ **ink/** | 96 | Ink 框架深度定制，事件系统/DOM 定制/ScrollBox/选择系统 |
| 🌉 **bridge/** | 31 | Bridge 远程控制协议分析 |

### 模块文档（28 份）

| 类别 | 模块 |
|------|------|
| 🏗️ 基础设施 | bootstrap · state · context · types · constants · entrypoints |
| 🖥️ 用户界面 | screens · ink · keybindings · vim · cli |
| 🧠 智能系统 | coordinator · memdir · services |
| 🔌 扩展系统 | plugins · skills · schemas · outputStyles |
| 🌐 远程网络 | remote · server · bridge · upstreamproxy |
| 🎨 特色功能 | buddy · voice · tasks · moreright · assistant |

## 🏗️ 架构总览

```
main.tsx (入口)
  ├── setup.ts → 环境初始化
  ├── commands.ts → 67+ 斜杠命令 (懒加载)
  ├── tools.ts → 33+ AI 工具 (权限分级)
  ├── QueryEngine.ts → LLM 查询循环
  │     └── query.ts → API 调用 + 工具执行
  ├── components/ → 439 个 Ink UI 组件
  │     ├── App.tsx → 顶层应用壳
  │     ├── PromptInput/ → 智能输入框
  │     ├── messages/ → 消息渲染系统
  │     └── permissions/ → 权限 UI
  ├── hooks/ → 104 个 React Hooks
  ├── utils/ → 564 个工具函数
  │     ├── permissions/ → 权限策略引擎 (6 种模式)
  │     ├── config/ → 多级配置系统 (7 种来源)
  │     └── model/ → 模型选择与降级
  ├── coordinator/ → 多 Agent 编排
  ├── services/ → API · MCP · OAuth · 语音 · 记忆
  └── state/ → 全局状态管理
```

## 📖 推荐阅读路径

### ⚡ 快速入门（2 小时）

```
背景知识 → main.tsx → QueryEngine → tools/ → commands/
```

### 🎯 完整路径（2-3 天）

```
阅读指南 → 背景知识
  → 核心入口文件（18 份）
  → tools/ → commands/ → components/
  → hooks/ → utils/
  → 模块文档（按兴趣选读）
```

## 🚀 本地运行

```bash
# 克隆仓库
git clone https://github.com/9p6p/claude-code-tutorial.git
cd claude-code-tutorial

# 安装依赖
pip install mkdocs mkdocs-material

# 启动预览
mkdocs serve

# 访问 http://127.0.0.1:8000
```

## 📝 关于

本教程由 [tutorial-any-repo](https://github.com/TKONIY/tutorial-any-repo) skill 生成框架指导，通过深度源码阅读和 AI 辅助分析完成。

- **目标读者**：想深入了解 Claude Code 内部实现的开发者
- **源码版本**：Claude Code npm 发布包（通过 source map 公开暴露的 TypeScript 源码）
- **文档语言**：全中文
- **教程仓库**：[9p6p/claude-code-tutorial](https://github.com/9p6p/claude-code-tutorial)

## 📄 许可

本教程仅供学习参考。Claude Code 源码版权归 [Anthropic](https://www.anthropic.com/) 所有。
