# `ink/` — 终端渲染引擎（96 个文件）

## 模块概述

`ink/` 是对开源 [Ink](https://github.com/vadimdemedes/ink) 的**大规模深度定制**（fork + 重写），已远远超出原始 Ink 的能力。它本质上是**在终端中运行的类浏览器 DOM + 渲染管道**。

## 架构层级

```
事件系统 (events/)     → capture/bubble 两阶段分发
  ↓
DOM 层 (dom.ts)        → 自定义节点 + 滚动 + 焦点
  ↓
协调器 (reconciler.ts) → React Reconciler 定制
  ↓
布局引擎 (layout/)     → Yoga WASM 布局
  ↓
渲染管道              → renderer → render-node-to-output → output → screen → log-update
  ↓
终端协议 (termio/)     → CSI/OSC/SGR/DEC 序列
```

## 核心文件深度分析

### `dom.ts` — 自定义 DOM 实现

**新增节点类型**：`ink-link`、`ink-progress`、`ink-raw-ansi`

**关键扩展**：
- **完整滚动系统**：`scrollTop`、`pendingScrollDelta`、`scrollClampMin/Max`、`scrollHeight`、`stickyScroll`、`scrollAnchor`
- **事件系统**：`_eventHandlers` 独立于 `attributes` 存储（避免 handler 变更触发 dirty）
- **焦点管理**：root 持有 `focusManager`
- **调试工具**：`debugOwnerChain` 记录 React 组件栈

### `reconciler.ts` — React Reconciler

- 事件处理器分离：`EVENT_HANDLER_PROPS` 存 `_eventHandlers` 而非 `attributes`
- Fiber 调试链：`getOwnerChain(fiber)` 追溯组件栈
- Yoga 节点清理：先清引用再释放 WASM 内存

### `ScrollBox.tsx` — 虚拟滚动

**绕过 React**：`scrollTo/scrollBy` 直接修改 DOM 节点的 `scrollTop`，标记 dirty + `scheduleRender`，无 React 开销。

- `scrollToElement`：延迟读取 `yogaNode.getComputedTop()`
- `setClampBounds`：虚拟滚动钳制，防止快速滚动越过 React 渲染

### `events/` — 完整事件系统（8 个文件）

**Claude Code 对标准 Ink 的最大架构新增**：

| 文件 | 说明 |
|------|------|
| `terminal-event.ts` | 完整的浏览器 Event API 镜像（target, currentTarget, eventPhase, stopPropagation） |
| `click-event.ts` | 鼠标点击：col, row, localCol, localRow, cellIsBlank |
| `keyboard-event.ts` | 键盘：key, ctrl, shift, meta, superKey, fn |
| `focus-event.ts` | 焦点：relatedTarget（类浏览器 focusin/focusout） |
| `dispatcher.ts` | **核心调度器**：capture/bubble 两阶段 + React 调度优先级映射 |

**Dispatcher 事件优先级**：键盘/点击/焦点 = `DiscreteEventPriority`，resize/scroll = `ContinuousEventPriority`

### `screen.ts` — 屏幕缓冲区（48KB）

- **CharPool / StylePool / HyperlinkPool**：字符串 interning 池，用 ID 引用
- **TypedArray 存储**：`Int32Array` 存 cell 数据，O(1) 访问 + 整数比较 diff
- **ASCII 快速路径**：ASCII 字符直接数组索引（不经 Map）
- **Blit 操作**：ID 跨 Screen 有效（pool 共享）

### `selection.ts` — 文本选择（34KB）

**完全从零构建**（标准 Ink 无此功能）：
- Anchor + Focus 模型，支持正向/反向选择
- 字符 / 词 / 行三种选择模式
- 拖拽滚动：`scrolledOffAbove/Below` 累加器
- Soft-wrap 感知

### `focus.ts` — 焦点管理器

- 焦点栈（最大 32 层）+ 去重
- 节点移除时自动从栈中恢复
- 支持点击聚焦和自动聚焦

### `terminal.ts` — 终端能力检测

- **进度报告**：OSC 9;4（ConEmu / Ghostty 1.2+ / iTerm2 3.6.6+）
- **同步输出**：DEC 2026（iTerm, WezTerm, Ghostty, VS Code, Alacritty, kitty）
- **tmux 检测**：明确跳过（不实现 DEC 2026）

### `stringWidth.ts` — 字符宽度

三层快速路径：
1. 纯 ASCII → 直接计数
2. 无 emoji → 逐字符 `eastAsianWidth`
3. 完整路径 → `Intl.Segmenter` grapheme 分段

### `ink.tsx` — 主控类（246KB）

所有子系统的中枢：双缓冲渲染、Pool 回收（每 5 分钟）、Alt-screen 管理、选区叠加、搜索高亮、光标管理、帧计时。

### `termio/` — 终端协议实现（8 个文件）

| 文件 | 内容 |
|------|------|
| `csi.ts` | CSI 序列（光标移动、Kitty 键盘协议） |
| `osc.ts` | OSC 序列（超链接、剪贴板、进度报告） |
| `dec.ts` | DEC 私有模式（Alt-Screen、鼠标跟踪） |
| `sgr.ts` | SGR 序列（颜色、样式解析） |
| `parser.ts` | 终端输出解析器 |
| `tokenize.ts` | 输入流分词 |

## 总结

`ink/` 展示了 Claude Code 对终端 UI 的极致追求——深度定制了滚动、事件（完整 capture/bubble）、焦点、文本选择、搜索高亮、终端协议等核心系统，在终端中实现了接近浏览器级别的交互能力。
