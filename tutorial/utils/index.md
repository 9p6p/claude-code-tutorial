# `utils/` — 工具函数库（564 文件）

## 模块概述

`utils/` 是项目**最大的模块**（564 个文件），包含了所有共享工具函数——从权限系统到 Git 操作，从配置管理到 Shell 解析。可以把它看作 Claude Code 的"工具箱"。

## 子目录索引

| 子目录 | 文件数 | 职责 |
|--------|--------|------|
| `permissions/` | 24 | **权限系统核心**（规则、分类器、模式管理） |
| `settings/` | 4+ | 多源配置合并框架 |
| `model/` | 8 | 模型选择、提供商、能力配置 |
| `bash/` | 2 | **纯 TypeScript Bash 解析器**（127KB！） |
| `shell/` | 1+ | Shell provider（bashProvider） |
| `git/` | 3 | Git 文件系统操作（配置解析、gitignore） |
| `swarm/` | 14 | **多 Agent 协作**（Tmux/进程内 swarm） |
| `computerUse/` | 5+ | 计算机操作 |
| `deepLink/` | 2+ | 深度链接处理 |
| `teleport/` | 1+ | Git bundle 传输 |
| `sandbox/` | 1+ | 沙箱隔离 |
| `telemetry/` | 3+ | 遥测 |
| `todo/` | 1+ | Todo 系统 |
| `suggestions/` | 3+ | 输入建议 |
| 根目录文件 | ~280+ | 众多独立工具模块 |

## 核心子系统深度分析

### 1. 权限系统 (`permissions/`) — 24 个文件

**核心文件**：

| 文件 | 大小 | 说明 |
|------|------|------|
| `permissions.ts` | **51KB** | 权限检查引擎（16 个导出函数） |
| `yoloClassifier.ts` | **51KB** | YOLO 自动审批 AI 分类器 |
| `permissionSetup.ts` | 52KB | 权限设置 UI 和初始化 |
| `filesystem.ts` | 61KB | 文件系统权限检查 |
| `PermissionMode.ts` | 3.4KB | 权限模式定义 |

**6 种权限模式**：

| 模式 | 说明 | 风险等级 |
|------|------|----------|
| `default` | 标准模式，工具调用需用户确认 | 🟢 安全 |
| `plan` | 规划模式，AI 只规划不执行 | 🟢 安全 |
| `acceptEdits` | 自动批准文件编辑操作 | 🟡 中等 |
| `bypassPermissions` | 绕过所有权限检查 | 🔴 危险 |
| `dontAsk` | 不询问直接拒绝/执行 | 🔴 危险 |
| `auto` | AI 分类器自动判断（仅内部用户） | 🟡 中等 |

**权限检查流程** (`checkRuleBasedPermissions`)：

```
1a. 工具是否被 deny 规则完全拒绝
1b. 工具是否有 ask 规则（需询问）
    - 特殊: Bash 工具在沙箱模式下可自动放行
1c. 工具自身的 checkPermissions()（如 Bash 子命令规则）
1d. 工具实现返回 deny 结果
```

**规则来源** (7 个)：`userSettings` · `projectSettings` · `localSettings` · `flagSettings` · `policySettings` · `cliArg` · `command` · `session`

**YOLO 分类器** (`yoloClassifier.ts`)：
- 使用 AI 模型作为"分类器"，自动判断工具调用是否安全
- 三组可自定义规则：`allow`（允许）、`soft_deny`（软拒绝）、`environment`（环境描述）
- 通过 `sideQuery` 旁路查询，不影响主对话流
- 支持 dump 到临时目录用于调试

### 2. 设置系统 (`settings/`) — 多源配置合并

**配置来源优先级（从低到高）**：

```
1. Plugin Settings — 最低优先级基础层
2. Policy Settings — 管理员策略，内部有独立优先级链：
   远程管理 → HKLM/macOS plist → managed-settings.json → HKCU
3. User Settings — ~/.claude/settings.json
4. Project Settings — .claude/settings.json
5. Local Settings — .claude/settings.local.json
6. Flag Settings — CLI --settings 标志指定的文件
```

**关键特性**：
- 使用 `lodash-es/mergeWith` 进行深度合并
- 文件解析结果缓存 + 会话级缓存 + Zod schema 验证
- 递归保护（`isLoadingSettings` 标志）
- 支持 `cowork_settings.json` 协作模式

### 3. 模型系统 (`model/`) — 8 个文件

**模型选择优先级**：

```
1. /model 命令的会话内覆盖
2. --model CLI 启动标志
3. ANTHROPIC_MODEL 环境变量
4. settings.json 中的 model 配置
5. 内置默认值（按用户类型差异化）
```

**默认模型分配**：

| 用户类型 | 默认模型 |
|---------|---------|
| 内部用户（ant） | Opus + [1m] |
| Max 订阅 | Opus (+1m) |
| Team Premium | Opus (+1m) |
| Pro/PAYG/Enterprise | Sonnet |

**4 种 API 提供商**：`firstParty`（直连 Anthropic）· `bedrock`（AWS）· `vertex`（GCP）· `foundry`（Azure）

### 4. Bash 解析器 (`bash/bashParser.ts`) — 128KB 巨型文件

一个**完整的纯 TypeScript Bash 解析器**，生成 tree-sitter-bash 兼容的 AST：

- **无外部依赖**：完全 TypeScript 实现，无 WASM
- **AST 节点**：`TsNode = { type, text, startIndex, endIndex, children }`
- **安全保护**：50ms 墙钟超时 + 最多 50,000 节点
- **词法器**：支持 16 种 Token 类型（WORD、DQUOTE、HEREDOC 等）
- **用途**：被权限系统用于解析 Bash 命令进行安全检查，是安全关键组件

### 5. 消息系统 (`messages.ts`) — 189KB

**项目中数一数二的巨型文件**，包含：
- 30+ 种消息类型的处理
- `normalizeMessagesForAPI()` — 内部格式 → API 格式转换
- `ensureToolResultPairing()` — 工具调用和结果正确配对
- 消息工厂：`createUserMessage()` 等
- 被几乎所有模块引用，是消息流转的核心

### 6. 配置系统 (`config.ts`) — 62KB

- 全局配置：`~/.claude/` 目录
- 项目配置：项目根 `.claude/` 下
- 再入保护：`insideGetConfig` 防无限递归
- 文件监控 + 锁文件防并发

### 7. 认证系统 (`auth.ts`) — 64KB

支持 6 种认证方式：
1. **API Key**：`ANTHROPIC_API_KEY`
2. **OAuth**：Claude AI OAuth 2.0 + PKCE
3. **AWS STS**：Bedrock 凭证
4. **GCP**：Vertex AI 凭证
5. **File Descriptor**：通过 FD 传递凭证
6. **macOS Keychain**：安全存储

### 8. CLAUDE.md 系统 (`claudemd.ts`) — 45KB

**加载优先级（从低到高）**：
1. `/etc/claude-code/CLAUDE.md` — 全局管理员指令
2. `~/.claude/CLAUDE.md` — 用户全局指令
3. `CLAUDE.md` + `.claude/rules/*.md` — 项目级指令
4. `CLAUDE.local.md` — 私有项目指令

**@include 指令**：支持 `@path`、`@./relative`、`@~/home`、`@/absolute`，防循环引用

### 9. Swarm 协作 (`swarm/`) — 14 个文件

多 Agent 协作系统的核心：
- **inProcessRunner.ts** (52KB)：进程内 Agent 运行器
- `AsyncLocalStorage` 实现上下文隔离
- 进度追踪、Plan mode 审批流
- 权限桥接 (`leaderPermissionBridge`)
- `TEAM_LEAD_NAME = 'team-lead'`
- 进程隔离的 socket 命名

## 关键设计模式

1. **编译时特性门控**：`feature('TRANSCRIPT_CLASSIFIER')` 用于死代码消除
2. **多层缓存策略**：文件解析缓存 → 会话缓存 → 运行时缓存
3. **闭包状态封装**：多处使用工厂函数+闭包代替类
4. **递归/再入保护**：`insideGetConfig`、`isLoadingSettings` 等守卫
5. **用户类型差异化**：`USER_TYPE === 'ant'` 内部用户享有更多功能

## 其他重要工具文件

| 文件 | 大小 | 说明 |
|------|------|------|
| `sessionStorage.ts` | 176KB | 会话持久化（最大文件之一） |
| `hooks.ts` | 156KB | 用户自定义 Hook 系统 |
| `teleport.tsx` | 172KB | Git bundle 传输（远程会话） |
| `attachments.ts` | 124KB | 附件处理（文件、图片、记忆） |
| `ansiToPng.ts` | 210KB | ANSI → PNG 转换 |
| `worktree.ts` | 49KB | Git Worktree 管理 |
| `status.tsx` | 48KB | /status 命令实现 |
| `Cursor.ts` | 46KB | 光标操作（Emacs kill ring） |
| `ripgrep.ts` | 21KB | ripgrep 集成 |
| `git.ts` | 30KB | Git 仓库检测和操作 |

## 总结

564 个文件的 `utils/` 是项目的"工具箱"——权限系统（24 文件，102KB）和设置管理是最复杂的子系统，Bash 解析器（128KB 纯 TS）是安全关键组件，消息处理（189KB）和配置管理（62KB）被全局依赖。
