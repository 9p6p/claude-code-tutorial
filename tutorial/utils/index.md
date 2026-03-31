# `utils/` — 工具函数库

## 模块概述

`utils/` 是项目最大的模块（564 个文件），包含了所有共享工具函数。从权限系统到 Git 操作，从配置管理到 Shell 解析。

## 关键子目录

| 子目录 | 说明 |
|--------|------|
| `permissions/` | 权限系统核心（PermissionMode、规则匹配、yolo 分类器） |
| `settings/` | 设置管理（多源配置合并、校验） |
| `model/` | 模型管理（选择、别名、价格、提供商） |
| `plugins/` | 插件管理（加载、缓存、市场） |
| `hooks/` | Hook 工具（事件处理、文件监听） |
| `bash/` | Bash 解析（命令拆分、AST、heredoc） |
| `shell/` | Shell 工具（提供商、输出限制、PowerShell） |
| `swarm/` | 团队协作（成员管理、邮箱、重连） |
| `sandbox/` | 沙箱隔离 |
| `telemetry/` | 遥测 |
| `task/` | 任务框架 |
| `todo/` | Todo 系统 |
| `suggestions/` | 输入建议 |

## 关键工具文件

| 文件 | 说明 |
|------|------|
| `messages.ts` | 消息处理（核心！归一化、分组、格式化） |
| `config.ts` | 配置读写（全局/项目） |
| `auth.ts` | 认证（OAuth、API Key、Bedrock/Vertex） |
| `git.ts` | Git 操作 |
| `claudemd.ts` | CLAUDE.md 加载 |
| `fileStateCache.ts` | 文件状态缓存 |
| `ripgrep.ts` | ripgrep 集成 |
| `imageResizer.ts` | 图片处理 |
| `pdf.ts` | PDF 读取 |
| `Cursor.ts` | 光标操作（Vim 用） |
| `format.ts` | 格式化工具 |
| `errors.ts` | 错误处理 |

## 总结

564 个文件的 `utils/` 是项目的"工具箱"。权限系统和设置管理是最复杂的子系统，各自有十几个文件的深度实现。
