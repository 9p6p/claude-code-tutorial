# `services/` — 核心服务层

## 模块概述

`services/` 是项目最大且最复杂的模块——包含 18 个子目录和 18 个顶层文件，覆盖了 API 通信、MCP 协议、OAuth 认证、语音识别、记忆系统等所有核心业务逻辑。

## 子目录索引

| 子目录 | 职责 |
|--------|------|
| `api/` | Claude API 调用层 |
| `mcp/` | Model Context Protocol 客户端 |
| `oauth/` | OAuth 认证流程 |
| `analytics/` | 分析/遥测（GrowthBook A/B 测试） |
| `compact/` | 上下文压缩 |
| `lsp/` | Language Server Protocol 管理 |
| `plugins/` | 插件服务 |
| `policyLimits/` | 策略限制（企业版） |
| `remoteManagedSettings/` | 远程托管设置 |
| `SessionMemory/` | 会话记忆管理 |
| `extractMemories/` | 后台记忆提取 |
| `autoDream/` | 自动梦境（记忆整理） |
| `teamMemorySync/` | 团队记忆同步 |
| `tips/` | 使用提示 |
| `AgentSummary/` | Agent 摘要 |
| `MagicDocs/` | 自动文档 |
| `PromptSuggestion/` | 提示建议 |
| `toolUseSummary/` | 工具使用摘要 |

## 顶层文件

| 文件 | 说明 |
|------|------|
| `voice.ts` + `voiceStreamSTT.ts` | 语音识别（STT） |
| `claudeAiLimits.ts` | 速率限制管理 |
| `tokenEstimation.ts` | Token 估算 |
| `vcr.ts` | API 录制/回放（测试用） |
| `notifier.ts` | OS 通知 |
| `preventSleep.ts` | 阻止系统休眠 |

## 总结

`services/` 是整个项目的"业务逻辑心脏"——如果说 `state/` 管理数据，`screens/` 管理 UI，那 `services/` 就是执行实际工作的地方。
