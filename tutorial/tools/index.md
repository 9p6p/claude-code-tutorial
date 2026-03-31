# `tools/` — AI 工具实现大全

## 模块概述

`tools/` 是 Claude Code 所有 **AI 工具的实现代码**，共 184 个文件、30+ 个工具目录。这些是 Claude AI 模型实际调用的工具——读文件、执行命令、搜索代码、创建子代理等。

## 工具分类索引

### 一、文件系统操作

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **FileReadTool** | `FileReadTool/` | 只读 | 读取文件，支持文本/图片/PDF/Notebook |
| **FileWriteTool** | `FileWriteTool/` | 需审批 | 创建或覆写文件 |
| **FileEditTool** | `FileEditTool/` | 需审批 | 精确查找替换编辑（非全量覆写） |
| **NotebookEditTool** | `NotebookEditTool/` | 需审批 | 编辑 Jupyter Notebook 单元格 |

### 二、搜索与发现

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **GlobTool** | `GlobTool/` | 只读 | glob 模式文件名搜索（限 100 结果） |
| **GrepTool** | `GrepTool/` | 只读 | 基于 ripgrep 的正则内容搜索 |
| **LSPTool** | `LSPTool/` | 只读 | 9 种 LSP 操作（定义跳转、引用查找等） |
| **ToolSearchTool** | `ToolSearchTool/` | 只读 | 搜索可用工具 |

### 三、命令执行

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **BashTool** | `BashTool/` (14 文件!) | 多层安全 | Shell 命令执行，最复杂的工具 |
| **PowerShellTool** | `PowerShellTool/` | 多层安全 | Windows PowerShell 执行 |

### 四、Agent/子代理

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **AgentTool** | `AgentTool/` (12 文件) | 按类型 | 创建子代理，6 种内置代理 |
| **SkillTool** | `SkillTool/` | 按规则 | 执行 Skills（/commit, /review 等） |

### 五、Web 工具

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **WebFetchTool** | `WebFetchTool/` | 按域名 | 抓取 URL 并转 markdown |
| **WebSearchTool** | `WebSearchTool/` | 需审批 | Anthropic Web Search API |

### 六、计划模式

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **EnterPlanModeTool** | `EnterPlanModeTool/` | 无需 | 进入规划模式 |
| **ExitPlanModeV2Tool** | `ExitPlanModeTool/` | 需审批 | 退出规划模式，提交计划审批 |

### 七、任务管理

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **TodoWriteTool** | `TodoWriteTool/` | 自动 | V1 任务检查清单 |
| **TaskCreateTool** | `TaskCreateTool/` | 自动 | V2 创建任务 |
| **TaskUpdateTool** | `TaskUpdateTool/` | 自动 | V2 更新任务 |
| **TaskGetTool** | `TaskGetTool/` | 自动 | V2 获取任务详情 |
| **TaskListTool** | `TaskListTool/` | 自动 | V2 列出任务 |
| **TaskStopTool** | `TaskStopTool/` | 需审批 | 停止运行中的任务 |

### 八、协作工具

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **TeamCreateTool** | `TeamCreateTool/` | 需审批 | 创建多 Agent 团队 |
| **TeamDeleteTool** | `TeamDeleteTool/` | 需审批 | 删除团队 |
| **SendMessageTool** | `SendMessageTool/` | 按类型 | Agent 间消息传递（含广播） |

### 九、其他工具

| 工具 | 目录 | 权限 | 说明 |
|------|------|------|------|
| **ConfigTool** | `ConfigTool/` | 读自动/写审批 | 读写 Claude Code 设置 |
| **AskUserQuestionTool** | `AskUserQuestionTool/` | 自动 | 向用户展示多选题 |
| **BriefTool** | `BriefTool/` | 按门控 | 后台模式下发送简报 |
| **EnterWorktreeTool** | `EnterWorktreeTool/` | 需审批 | 创建 Git worktree 隔离环境 |
| **ExitWorktreeTool** | `ExitWorktreeTool/` | 需审批 | 退出 worktree |
| **MCPTool** | `MCPTool/` | passthrough | MCP 工具基础框架 |
| **CronCreateTool** | `ScheduleCronTool/` | 需审批 | 创建定时任务 |
| **RemoteTriggerTool** | `RemoteTriggerTool/` | 需认证 | 管理远程触发器 |

## BashTool — 最复杂的工具

BashTool 是 14 个文件、总计 300KB+ 的"巨型工具"：

| 文件 | 大小 | 职责 |
|------|------|------|
| `bashSecurity.ts` | 100KB | 命令安全分析（防注入） |
| `bashPermissions.ts` | 96KB | 权限规则匹配 |
| `readOnlyValidation.ts` | 67KB | 只读模式验证 |
| `pathValidation.ts` | 43KB | 路径安全检查 |
| `sedValidation.ts` | 21KB | sed 命令安全验证 |
| `prompt.ts` | 21KB | Git 操作提示词 |

安全层实现**纵深防御**：命令解析 → 危险模式检测 → 路径验证 → 权限规则 → 用户审批。

## AgentTool — 子代理系统

12 个文件，6 种内置代理：

| 内置代理 | 用途 |
|---------|------|
| GeneralPurpose | 通用任务 |
| Explore | 代码库探索 |
| Plan | 任务规划 |
| Verification | 变更验证 |
| ClaudeCodeGuide | 使用指南 |
| StatuslineSetup | 状态栏配置 |

## 核心架构模式

所有工具通过 `buildTool()` 工厂函数构建，统一接口：
- `inputSchema`：Zod 验证输入
- `checkPermissions()`：权限检查
- `validateInput()`：输入验证
- `call()`：执行逻辑
- `renderToolUseMessage()`：UI 渲染

## 总结

`tools/` 是 Claude Code "动手做事"的能力来源。33+ 工具覆盖了文件操作、代码搜索、命令执行、子代理创建、任务管理等所有 AI 编程场景。BashTool 的 300KB 安全层展示了对安全性的极致追求。
