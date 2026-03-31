# Claude Code 源码教程

> 📖 对 [Claude Code](https://github.com/anthropics/claude-code) 项目的深度逐文件代码教程，覆盖 43 个模块、512,000+ 行 TypeScript 源码。

## 🌐 在线阅读

👉 **[https://9p6p.github.io/claude-code-tutorial/](https://9p6p.github.io/claude-code-tutorial/)**

## ✨ 特性

- 📚 **46 份教程文档**，全中文
- 🎨 **Material 主题**，支持亮色/暗色模式切换
- 🔍 **中文全文搜索**
- 📊 **架构图和流程图**
- 💻 **代码高亮** + 一键复制

## 📁 文档覆盖

### 核心入口文件（18 份）
`main.tsx` · `commands.ts` · `tools.ts` · `Tool.ts` · `QueryEngine.ts` · `query.ts` · `context.ts` · `setup.ts` · `ink.ts` · `Task.ts` · `tasks.ts` · `history.ts` · `cost-tracker.ts` · `costHook.ts` · `replLauncher.tsx` · `dialogLaunchers.tsx` · `interactiveHelpers.tsx` · `projectOnboardingState.ts`

### 模块文档（23 份）

| 类别 | 模块 |
|------|------|
| 🏗️ 基础设施 | bootstrap · state · context · types · constants · entrypoints |
| 🖥️ 用户界面 | screens · keybindings · vim · cli |
| 🧠 智能系统 | coordinator · memdir · services |
| 🔌 扩展系统 | plugins · schemas · outputStyles |
| 🌐 远程网络 | remote · server · upstreamproxy |
| 🎨 特色功能 | buddy · voice · moreright · assistant |

## 🏗️ 架构总览

```
main.tsx (入口)
  ├── setup.ts → 环境初始化
  ├── commands.ts → 60+ 斜杠命令
  ├── tools.ts → 30+ AI 工具
  ├── QueryEngine.ts → LLM 查询循环
  │     └── query.ts → API 调用 + 工具执行
  ├── replLauncher.tsx → REPL 启动
  │     └── screens/REPL.tsx → 主界面 (875KB!)
  ├── coordinator/ → 多 Agent 编排
  ├── services/ → API · MCP · OAuth · 语音 · 记忆
  └── state/ → 全局状态管理
```

## 🚀 本地运行

```bash
# 安装依赖
pip install mkdocs mkdocs-material

# 启动预览
mkdocs serve

# 访问 http://127.0.0.1:8000
```

## 📝 关于

本教程由 [tutorial-any-repo](https://github.com/TKONIY/tutorial-any-repo) skill 生成框架指导，通过深度源码阅读和 AI 辅助分析完成。

目标读者：想深入了解 Claude Code 内部实现的开发者。

## 📄 许可

本教程仅供学习参考。Claude Code 源码版权归 [Anthropic](https://www.anthropic.com/) 所有。
