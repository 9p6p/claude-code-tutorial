# `ink/` — 终端渲染引擎

## 模块概述

`ink/` 是 Ink 框架的**修改版**（96 个文件），Claude Code 对官方 Ink 进行了大量定制以满足高性能终端 UI 需求。

## 关键定制

| 组件/功能 | 说明 |
|-----------|------|
| `ScrollBox` | 自定义滚动容器（虚拟滚动） |
| `Button` | 终端按钮组件 |
| `Link` | 可点击链接 |
| `use-input` | 输入处理 Hook |
| `use-selection` | 选择状态 Hook |
| `use-search-highlight` | 搜索高亮 |
| `use-terminal-focus` | 终端焦点检测 |
| `use-tab-status` | 标签状态 |
| `FocusManager` | 焦点管理 |
| `events/` | 自定义事件系统（InputEvent、ClickEvent） |
| `dom.ts` | 自定义 DOM 实现 |
| `terminal.ts` | 终端检测 |
| `stringWidth.ts` | 字符宽度计算 |
| `render-border.ts` | 边框渲染 |

## 总结

`ink/` 展示了 Claude Code 对终端 UI 的极致追求——不满足于 Ink 的默认能力，深度定制了滚动、事件、焦点等核心系统。
