# `components/` — UI 组件库（389 个文件）

## 模块概述

`components/` 构成了完整的终端 UI 应用层。所有组件都使用了 **React Compiler** 的输出格式（`_c()` memo cache），实现自动的渲染优化。

## 组件分类索引

| 类别 | 文件数 | 说明 |
|------|--------|------|
| App 层 | 2 | 顶层容器和全屏布局 |
| 消息系统 | 34 | 34 种消息类型组件 |
| 输入系统 | 20+ | PromptInput 及其子组件 |
| 权限系统 | 47 | 权限请求 UI |
| 设计系统 | 16 | 原语组件 |
| MCP | 14 | MCP 服务器管理 |
| Spinner | 12 | 加载动画系统 |
| 任务/团队 | 12 | 后台任务管理 |
| 设置 | 4 | 配置/状态/用量 |
| Agent | 12 | Agent 管理 UI |

## 超级组件 Top 5

| 组件 | 大小 | 说明 |
|------|------|------|
| `PromptInput.tsx` | **347KB** | 智能输入框（集成历史、补全、粘贴、Vim、语音等） |
| `ink/ink.tsx` | 246KB | Ink 主引擎 |
| `Messages.tsx` | 144KB | 消息列表管理器 |
| `ScrollKeybindingHandler.tsx` | 146KB | 滚动快捷键处理 |
| `MessageSelector.tsx` | 113KB | 消息选择器 |

## 核心组件分析

### `App.tsx` — 应用入口

纯包装器，三层 Provider 嵌套：`FpsMetricsProvider` → `StatsProvider` → `AppStateProvider`

### `PromptInput/` — 智能输入系统（347KB + 10 个子文件）

集成了几乎所有输入相关功能：
- 箭头键历史（`useArrowKeyHistory`）
- Typeahead 自动补全（`useTypeahead`）
- IDE @ 引用（`useIdeAtMentioned`）
- 输入缓冲区/撤销（`useInputBuffer`）
- Prompt 建议（`usePromptSuggestion`）
- 权限模式切换、图片粘贴、Vim 模式、Slack 频道建议等

### `Messages.tsx` — 消息列表（144KB）

**消息预处理管道**：
```
normalizeMessages → reorderMessagesInUI → applyGrouping 
→ collapseReadSearchGroups → collapseHookSummaries 
→ collapseTeammateShutdowns → collapseBackgroundBashNotifications
```

使用 `VirtualMessageList` 实现虚拟化滚动，支持 Brief mode 过滤。

### `messages/` — 34 种消息组件

| 类别 | 组件 |
|------|------|
| **Assistant** | TextMessage, ThinkingMessage, RedactedThinkingMessage, ToolUseMessage |
| **User** | TextMessage, ImageMessage, BashInputMessage, BashOutputMessage, CommandMessage, PlanMessage, TeammateMessage, ChannelMessage 等 |
| **System** | TextMessage, APIErrorMessage, RateLimitMessage, ShutdownMessage |
| **Tool** | GroupedToolUseContent, CollapsedReadSearchContent, HookProgressMessage |
| **附件** | AttachmentMessage (70KB，最大的消息组件) |

### `permissions/` — 权限系统（47 个文件）

通过工具类型分发到对应权限请求 UI：

```typescript
switch (tool) {
  case FileEditTool:  → FileEditPermissionRequest
  case BashTool:      → BashPermissionRequest
  case PowerShellTool → PowerShellPermissionRequest
  case WebFetchTool:  → WebFetchPermissionRequest
  case SkillTool:     → SkillPermissionRequest
  // ... 共 13 种
  default:            → FallbackPermissionRequest
}
```

### `design-system/` — 设计系统原语（16 个文件）

| 组件 | 说明 |
|------|------|
| `ThemedBox` | 主题感知 Box（`borderColor` 接受 Theme key 或原始颜色） |
| `ThemedText` | 主题感知 Text |
| `ThemeProvider` | 主题管理（'light'/'dark'/'auto'，OSC 11 探测） |
| `Dialog` | 确认/取消对话框（内置快捷键） |
| `Pane` | 终端区域面板 |
| `FuzzyPicker` | 模糊搜索选择器 |
| `Tabs` | 标签页组件（40KB） |
| `Divider` | 分割线 |
| `ProgressBar` | 进度条 |

### `mcp/` — MCP 服务器管理（14 个文件）

`MCPSettings` 使用状态机驱动：`list → server-menu → tool-list → tool-detail`

### `Spinner.tsx` — 加载动画（86KB）

支持 `TeammateSpinnerTree`（多 agent 状态树）、shimmer 动画、Brief mode 分支。

### `FullscreenLayout.tsx` — 全屏布局（83KB）

`ScrollChromeContext` + `useUnseenDivider` 实现 "N new messages" 气泡和 sticky header。

## 通用代码模式

1. **React Compiler**：`_c()` memo cache，自动跳过未变化的子树
2. **Feature flags**：`feature('KAIROS')` 等编译时常量
3. **条件 require**：`feature('X') ? require('...') : null`
4. **useAppState selector**：选择性订阅全局状态
5. **Theme 集成**：颜色值使用 `keyof Theme` 而非硬编码

## 总结

389 个组件构成了 Claude Code 完整的终端 UI——从 347KB 的智能输入框到 34 种消息类型、47 个权限组件、16 个设计系统原语，在终端中实现了接近 Web 应用级别的交互能力。
