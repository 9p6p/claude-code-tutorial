# `screens/` — 屏幕组件

## 模块概述

`screens/` 包含 Claude Code 的三个顶级屏幕组件，是用户直接看到的 UI 界面。

## 文件清单

| 文件 | 大小 | 说明 |
|------|------|------|
| `REPL.tsx` | **875KB** | 主 REPL 交互界面（项目最大文件！） |
| `Doctor.tsx` | 72KB | 诊断/健康检查界面 |
| `ResumeConversation.tsx` | 58KB | 恢复会话选择界面 |

## REPL.tsx — 项目的"心脏"

`REPL.tsx` 是整个 Claude Code 的**主 UI 组件**，一个 5,006 行、875KB 的"超级组件"。它集成了项目中几乎所有其他模块：

### 集成的 Hooks

```typescript
useRemoteSession()        // 远程会话
useDirectConnect()        // 直连模式
useSSHSession()          // SSH 会话
useVoiceIntegration()    // 语音集成
useMoreRight()           // 内部功能
useSearchInput()         // 搜索
useSwarmInitialization() // 团队协作
useCostSummary()         // 成本追踪
useSkillImprovementSurvey() // 技能改进调查
```

### 核心职责

1. **消息渲染**：显示用户和 AI 的对话
2. **输入处理**：接收用户输入（支持 Vim 模式）
3. **权限对话框**：显示工具权限确认
4. **工具执行 UI**：显示工具执行进度
5. **远程会话**：管理 CCR/SSH/DirectConnect 模式
6. **成本显示**：实时显示 token 消耗和费用

### 使用 React Compiler

```typescript
import { c as _c } from "react/compiler-runtime";
```

REPL.tsx 使用了 React Compiler 进行自动优化。

## Doctor.tsx

诊断界面，检查：
- Node.js 版本
- API Key 有效性
- 网络连接
- Git 配置
- 插件健康状态

## ResumeConversation.tsx

恢复已保存对话的选择界面，显示最近的对话列表并允许用户选择继续。

## 总结

`screens/` 是用户直接交互的 UI 层。REPL.tsx 虽然巨大，但作为终端 UI 的唯一入口，它承载了所有交互逻辑。
