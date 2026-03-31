# `services/` — 核心服务层（18 个子模块）

## 模块概述

`services/` 是整个项目的**业务逻辑心脏**——如果说 `state/` 管理数据，`screens/` 管理 UI，那 `services/` 就是执行实际工作的地方。包含 18 个子目录和 18 个顶层文件。

## 子模块深度索引

### 1. `api/` — API 客户端层（20 个文件）

| 文件 | 大小 | 说明 |
|------|------|------|
| `claude.ts` | **123KB** | 核心 API 调用层（消息规范化、缓存控制、Beta 管理） |
| `errors.ts` | 41KB | 错误处理（prompt_too_long 检测、重试判断） |
| `withRetry.ts` | 28KB | 重试逻辑（指数退避、断路器） |
| `client.ts` | 16KB | SDK 客户端工厂（4 种提供商） |
| `promptCacheBreakDetection.ts` | 26KB | Prompt Cache 中断检测 |
| `filesApi.ts` | 21KB | 文件 API（上传/下载） |
| `logging.ts` | 24KB | API 日志 |

**client.ts 支持 4 种提供商**：Direct API（`ANTHROPIC_API_KEY`）· AWS Bedrock · Google Vertex · Azure Foundry

### 2. `mcp/` — MCP 协议客户端（23 个文件）

| 文件 | 大小 | 说明 |
|------|------|------|
| `client.ts` | **116KB** | MCP 客户端核心 |
| `config.ts` | 50KB | MCP 服务器配置管理 |
| `auth.ts` | 87KB | MCP OAuth 认证 |
| `useManageMCPConnections.ts` | 44KB | React Hook 管理连接 |

**支持传输方式**：`StdioClientTransport` · `SSEClientTransport` · `StreamableHTTPClientTransport` · `WebSocketTransport`

### 3. `oauth/` — OAuth 2.0 认证（5 个文件）

实现 **OAuth 2.0 Authorization Code + PKCE** 流程：
- 自动模式：打开浏览器 → localhost 回调捕获码
- 手动模式：用户手动复制粘贴授权码

### 4. `compact/` — 对话压缩引擎（11 个文件）

| 文件 | 说明 |
|------|------|
| `compact.ts` (59KB) | 主压缩引擎，使用 fork agent 共享 prompt cache |
| `autoCompact.ts` (13KB) | 自动触发（上下文窗口 - 20,000 输出保留） |
| `microCompact.ts` (19KB) | 微压缩（更轻量的上下文缩减） |
| `sessionMemoryCompact.ts` (21KB) | 会话记忆压缩 |

### 5. `SessionMemory/` — 会话记忆服务（3 个文件）

自动维护 Markdown 文件，记录当前对话关键信息。后台定期使用 fork 子代理运行，通过 `registerPostSamplingHook` 注册。

### 6. `extractMemories/` — 记忆提取（2 个文件）

从会话转录中提取持久化记忆，写入 `~/.claude/projects/<path>/memory/`。在每次完整查询循环结束时运行。

### 7. `autoDream/` — 自动梦境/整合（4 个文件）

类似人类睡眠的记忆巩固——后台运行 `/dream` 提示词整合记忆。

**门控逻辑（从低成本到高成本）**：
1. 时间门：距上次 ≥24 小时
2. 会话门：≥5 个新会话
3. 锁门：无其他进程正在整合

### 8. `lsp/` — 语言服务（7 个文件）

`LSPServerManager` 接口支持：`initialize` · `shutdown` · `getServerForFile` · `sendRequest` · `openFile/changeFile/saveFile/closeFile`

全局单例，初始化状态机：`'not-started' | 'pending' | 'success' | 'failed'`

### 9. `voice.ts` — 语音输入（17KB）

按键说话录音。**录音后端优先级**：
1. 原生音频捕获（cpal）：macOS/Linux/Windows
2. SoX `rec`：Linux 后备
3. arecord (ALSA)：Linux 后备

16kHz 采样率、单声道。延迟加载 `audio-capture-napi`。

### 10. `vcr.ts` — API Fixture 录制/回放（12KB）

类 VCR 模式：`withFixture(input, fixtureName, fn)` → SHA-1 哈希 → 首次执行缓存 → 后续直接返回

### 顶层服务文件

| 文件 | 说明 |
|------|------|
| `claudeAiLimits.ts` (16KB) | 速率限制管理 |
| `tokenEstimation.ts` (16KB) | Token 估算 |
| `notifier.ts` (4KB) | OS 桌面通知 |
| `preventSleep.ts` (4.5KB) | 阻止系统休眠 |
| `diagnosticTracking.ts` (12KB) | 诊断追踪 |

## 架构亮点

**Fork Agent 模式**：compact、SessionMemory、extractMemories、autoDream 都使用分叉子代理——共享父级 prompt cache，不干扰主对话。

## 总结

`services/` 的 18 个子模块覆盖了 API 通信、MCP 协议、OAuth 认证、对话压缩、记忆系统、语音识别、LSP 集成等所有核心业务逻辑。
