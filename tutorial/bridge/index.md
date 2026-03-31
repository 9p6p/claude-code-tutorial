# `bridge/` — Bridge 远程控制

## 模块概述
`bridge/` 实现了 Claude Code 的 **Bridge 模式**（31 个文件）——允许通过 claude.ai 远程控制本地终端中运行的 Claude Code。

## 核心功能
- WebSocket 双向通信
- 权限请求桥接
- 心跳保活和自动重连
- 会话状态同步

---

# `skills/` — Skills 加载系统

## 模块概述
`skills/` 管理 Claude Code 的 **Skills 系统**（20 个文件）——从 `.claude/skills/` 目录加载用户自定义技能。

## 核心文件
| 文件 | 说明 |
|------|------|
| `loadSkillsDir.ts` | 从目录加载 Skills |
| `bundledSkills.ts` | 内置 Skills |
| `bundled/index.ts` | 内置 Skills 注册 |

---

# `tasks/` — 后台任务实现

## 模块概述
`tasks/` 包含 **6 种后台任务** 的具体实现（12 个文件）。

## 任务类型
| 任务 | 说明 |
|------|------|
| `LocalShellTask` | 本地 Shell 命令 |
| `LocalAgentTask` | 本地子 Agent |
| `RemoteAgentTask` | 远程代理 |
| `InProcessTeammateTask` | 进程内队友 |
| `DreamTask` | 后台推理 |
| `LocalWorkflowTask` | 本地工作流 |

---

# `migrations/` — 配置迁移

## 模块概述
`migrations/` 包含 11 个配置迁移脚本，处理 Claude Code 版本升级时的配置格式变更。

---

# `native-ts/` — 原生 TypeScript 工具

## 模块概述
`native-ts/` 包含 4 个文件，提供原生 TypeScript 运行时支持。

---

# `query/` — 查询辅助模块

## 模块概述
`query/` 包含 4 个文件，提供 API 查询的辅助功能（过滤、缓存策略等）。
