# `components/` — UI 组件库

## 模块概述

`components/` 是 Claude Code 的**完整终端 UI 层**，共 439 个文件。基于 Ink（终端 React 渲染器）构建，包含消息渲染、输入处理、权限对话框、设计系统等。

## 目录结构

| 子目录 | 文件数 | 说明 |
|--------|--------|------|
| `messages/` | 42 | 消息渲染（助手/用户/系统/工具） |
| `PromptInput/` | 21 | 输入系统（346KB 主组件！） |
| `permissions/` | 66 | 权限确认 UI |
| `design-system/` | 16 | 基础组件库 |
| `mcp/` | 14 | MCP 服务器管理 |
| `Spinner/` | 12 | 加载动画 |
| `LogoV2/` | 15 | Logo 和欢迎页 |
| `Settings/` | 4 | 设置面板（Config.tsx 265KB!） |
| `CustomSelect/` | 10 | 自定义选择控件 |
| `FeedbackSurvey/` | 9 | 反馈调查 |
| `agents/` | 14 | Agent 管理 |
| `tasks/` | 12 | 后台任务 UI |
| 顶层文件 | ~113 | 各种独立组件 |

## 超级组件 Top 5

| 组件 | 大小 | 说明 |
|------|------|------|
| `PromptInput/PromptInput.tsx` | **347KB** | 输入区（项目最大单文件！） |
| `Settings/Config.tsx` | **265KB** | 配置面板 |
| `LogSelector.tsx` | **196KB** | 日志选择器 |
| `mcp/ElicitationDialog.tsx` | **175KB** | MCP 信息采集 |
| `Stats.tsx` | **149KB** | 统计面板 |

## 核心组件链

```
App.tsx (根容器)
  └── FpsMetricsProvider → StatsProvider → AppStateProvider
        └── REPL.tsx (主屏幕)
              ├── Messages.tsx (144KB, 消息列表)
              │     └── MessageRow → Message → messages/*.tsx
              ├── Spinner.tsx (86KB, 加载状态)
              ├── PromptInput.tsx (347KB, 输入区)
              └── PermissionRequest.tsx (权限对话框)
```

## 设计系统组件

| 组件 | 说明 |
|------|------|
| `ThemeProvider` | 主题管理（dark/light/auto） |
| `ThemedBox/ThemedText` | 主题感知的基础组件 |
| `Dialog` | 确认/取消对话框 |
| `Tabs` | 标签页 |
| `FuzzyPicker` | 模糊搜索选择器 |
| `Pane` | 命令屏幕容器 |
| `Divider` | 分隔线 |
| `ProgressBar` | 进度条 |

## 全局性设计模式

1. **React Compiler**：几乎所有组件都经 React Compiler 编译（`_c()` 缓存）
2. **编译时 DCE**：`feature()` 实现死代码消除
3. **极致性能**：模块级缓存、WeakMap diff 缓存、OffscreenFreeze、虚拟滚动
4. **Ink 终端 UI**：基于 Box/Text 而非 DOM

## 总结

`components/` 是 Claude Code 的"脸面"——439 个组件构成了完整的终端 UI 体验。从 347KB 的 PromptInput 到 16 个设计系统组件，展示了终端 UI 可以做到和 Web UI 一样丰富和精致。
