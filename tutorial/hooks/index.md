# `hooks/` — React Hooks 集合（104 个文件）

## 模块概述

`hooks/` 包含 **104 个 React Hook 文件**，是连接 UI 组件和业务逻辑的桥梁。分为三个子目录：

| 目录 | 文件数 | 说明 |
|------|--------|------|
| 主目录 | ~82 | 核心业务 Hooks |
| `notifs/` | ~18 | 通知相关 Hooks |
| `toolPermission/` | ~4 | 工具权限处理器 |

## 核心 Hooks 深度分析

### 1. `useCanUseTool` (39KB) — 工具权限检查

**最核心的 Hook**——每次 AI 要调用工具时执行。

**决策链**：
```
hasPermissionsToUseTool()
  ├── allow → 直接放行（auto-mode 记录分类器审批）
  ├── deny → 记录拒绝（auto-mode 显示通知）
  └── ask → 多层处理：
       1. handleCoordinatorPermission（协调器权限）
       2. handleSwarmWorkerPermission（团队成员权限）
       3. peekSpeculativeClassifierCheck（投机分类器检查）
       4. handleInteractivePermission（用户交互确认）
```

**返回类型**：`CanUseToolFn` — async 函数，返回 `Promise<PermissionDecision<Input>>`

### 2. `useRemoteSession` (22KB) — 远程会话管理

**管理的状态**：WebSocket 连接状态、远程后台任务计数、响应超时检测

```typescript
type UseRemoteSessionResult = {
  isRemoteMode: boolean
  sendMessage: (content, opts?) => Promise<boolean>
  cancelRequest: () => void
  disconnect: () => void
}
```

**关键设计**：
- 双通道：WebSocket 接收消息流，HTTP POST 发送用户输入
- `BoundedUUIDSet(50)` 去重已发送消息
- 超时检测：常规 60s，压缩期间 180s

### 3. `useVoiceIntegration` (97KB) — 语音集成

**最大的 Hook 文件**，实现按住说话（Push-to-Talk）：

| 参数 | 值 | 说明 |
|------|-----|------|
| `RAPID_KEY_GAP_MS` | 120ms | 按键是否为持续按下 |
| `HOLD_THRESHOLD` | 5 | 裸键需连续 5 次快速按键才激活 |
| `WARMUP_THRESHOLD` | 2 | 2 次开始显示预热反馈 |
| `MODIFIER_FIRST_PRESS_FALLBACK_MS` | 2000ms | 修饰键组合首次按下即激活 |

默认语音快捷键为空格键。通过 `feature('VOICE_MODE')` 编译时门控。

### 4. `useSearchInput` (10KB) — 搜索输入

完整的文本编辑快捷键支持：
- **Emacs kill ring**：`Ctrl+K/U/W` 剪切, `Ctrl+Y` 粘贴, `Meta+Y` yank-pop
- 过滤特殊按键（pageup/pagedown/insert/mouse/F1-F12）
- `backspaceExitsOnEmpty` 选项

### 5. `useTerminalSize` (354B) — 终端尺寸

极简封装（仅 15 行），从 `TerminalSizeContext` 读取行/列信息。几乎所有需要终端尺寸的 UI 组件都依赖它。

### 6. `useInputBuffer` (3.3KB) — 输入缓冲/撤销

**BufferEntry**：`{ text, cursorOffset, pastedContents, timestamp }`

- 去抖动式记录编辑状态
- 超出 `maxBufferSize` 时截断最旧条目
- 在 undo 后新输入时截断当前位置之后的历史

### 7. `useArrowKeyHistory` (33KB) — 历史导航

- `HISTORY_CHUNK_SIZE = 10`：按块加载历史减少磁盘读取
- 批量加载优化：并发请求合并为单次磁盘读取
- 支持 `HistoryMode` 过滤（按输入模式筛选历史）

### 8. `useGlobalKeybindings` (30KB) — 全局快捷键

| 快捷键 | 功能 |
|--------|------|
| `Ctrl+T` | 切换任务列表（循环: none → tasks → teammates → none） |
| `Ctrl+O` | 切换转录模式 (prompt ↔ transcript) |
| `Ctrl+E` | 切换显示全部消息 |
| `Ctrl+Shift+B` | 切换 Brief-only 视图 |
| `Ctrl+C/Escape` | 退出转录模式 |

### 9. `useCancelRequest` (10KB) — 请求取消

**取消优先级**：
1. 如果有活跃任务（`abortSignal` 未中止），取消它
2. Claude 空闲时弹出命令队列
3. 清空工具确认队列 + 触发 onCancel

**双击杀死**：`KILL_AGENTS_CONFIRM_WINDOW_MS = 3000ms` 内连续 Ctrl+C 杀死所有后台 agent

### 10. `useBackgroundTaskNavigation` (8.4KB) — 后台任务导航

`Shift+Up/Down` 在 teammate 间循环导航：`leader(-1) → teammate(0..n-1) → hide(n) → leader...`

### 11. `useSwarmInitialization` (3.1KB) — Swarm 初始化

两种初始化路径：
1. **恢复会话**：从 transcript 消息中提取 `teamName` + `agentName`
2. **新 spawn**：从环境变量读取上下文

### 12. `useLogMessages` (5.6KB) — 消息日志

**增量记录优化**：只记录新消息尾部，避免 O(n) 全量扫描。检测三种场景：incremental、compaction、same-head-shrink。

### 13. `useDirectConnect` (7.3KB) — 直连模式

与 `useRemoteSession` 互补——前者连 CCR 云端，后者连自托管服务器。两者返回值形态一致。

### 14. `useApiKeyVerification` (2.6KB) — API Key 验证

状态机：`'loading' | 'valid' | 'invalid' | 'missing' | 'error'`

### 15. `useSettingsChange` (946B) — 设置变更监听

极简订阅模式，通过 `settingsChangeDetector.subscribe` 监听配置文件变化。

## 通知 Hooks (`notifs/`) — 18 个

| Hook | 说明 |
|------|------|
| `useFastModeNotification` | 快速模式通知 |
| `useRateLimitWarningNotification` | 速率限制警告 |
| `useLspInitializationNotification` | LSP 初始化通知 |
| `useMcpConnectivityStatus` | MCP 连接状态 |
| `useIDEStatusIndicator` | IDE 状态指示器 |
| `useSettingsErrors` | 设置文件错误 |
| `usePluginInstallationStatus` | 插件安装状态 |
| 等等... | |

## 总结

104 个 Hooks 覆盖了 Claude Code 的**全部运行时行为**：工具权限决策（多层 classifier + swarm 链）、远程会话管理（WebSocket + HTTP POST）、语音输入（97KB 的按住检测）、输入编辑（Emacs kill ring）、全局快捷键、请求取消（优先级链 + 双击杀 agent）、Swarm 管理。
