# `hooks/` — React Hooks 集合

## 模块概述

`hooks/` 包含 104 个 React Hook 文件，是连接 UI 组件和业务逻辑的桥梁。

## 关键 Hooks

| Hook | 说明 |
|------|------|
| `useCanUseTool` | 工具权限检查 |
| `useRemoteSession` | 远程 CCR 会话管理 |
| `useDirectConnect` | Direct Connect 模式 |
| `useSSHSession` | SSH 会话 |
| `useVoiceIntegration` | 语音集成 |
| `useSearchInput` | 搜索输入 |
| `useSwarmInitialization` | 团队初始化 |
| `useTerminalSize` | 终端尺寸 |
| `useInputBuffer` | 输入缓冲 |
| `useArrowKeyHistory` | 方向键历史导航 |
| `useApiKeyVerification` | API Key 验证 |
| `useGlobalKeybindings` | 全局快捷键 |
| `useCancelRequest` | 取消请求 |
| `useBackgroundTaskNavigation` | 后台任务导航 |
| `useLogMessages` | 日志消息 |
| `useSettingsChange` | 设置变更监听 |
| `useAfterFirstRender` | 首次渲染后回调 |
| `useSkillImprovementSurvey` | 技能改进调查 |

## 总结

104 个 Hooks 体现了 Claude Code 对 React Hook 模式的深度运用——几乎每个业务关注点都被封装为独立 Hook。
