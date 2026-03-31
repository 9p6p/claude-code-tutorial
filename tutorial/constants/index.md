# `constants/` — 常量与配置中心（21 个文件）

## 模块概述

`constants/` 是全应用范围的常量定义。其中 **`prompts.ts`（53KB）是最核心的文件**——它定义了 Claude Code 的"人格"，包含完整的系统提示词构建逻辑。

## 🧠 `prompts.ts` — 系统提示词（53KB）

### 系统提示的整体结构

由 `getSystemPrompt()` 函数组装，分为**静态可缓存部分**和**动态部分**：

```
[静态内容 — 跨组织可缓存]
  ├── Intro（身份 + 安全指令）
  ├── System（系统级规则）
  ├── Doing Tasks（任务执行规范）
  ├── Executing Actions with Care（安全操作）
  ├── Using Your Tools（工具使用指南）
  ├── Tone and Style（语气风格）
  └── Output Efficiency（输出效率）

=== SYSTEM_PROMPT_DYNAMIC_BOUNDARY ===

[动态内容 — 每会话/每轮变化]
  ├── Session-specific Guidance
  ├── Memory（CLAUDE.md）
  ├── Environment Info
  ├── Language Preference
  ├── Output Style
  ├── MCP Instructions
  ├── Scratchpad
  └── Numeric Length Anchors (Ant-only)
```

### 关键章节

**a) Intro**：身份定义 + `CYBER_RISK_INSTRUCTION`（安全风险指令，禁止 DoS/供应链攻击）+ 严禁猜测 URL

**b) Doing Tasks**（最小改动原则）：
- 不添加未请求的功能、不重构、不加多余注释
- 不为不可能的场景添加错误处理
- 不为假设需求设计
- Ant-only：默认不写注释、完成前必须验证、如实报告结果

**c) Using Your Tools**（工具使用指南）：
```
FileRead  代替 cat/head/tail
FileEdit  代替 sed/awk
FileWrite 代替 cat heredoc
Glob      代替 find/ls
Grep      代替 grep/rg
```

**d) Output Efficiency**：
- Ant 内部版：详细的"用户沟通"指南（流畅散文、避免 emoji）
- 外部版：简洁直接——"直入主题，先试最简单的方法"
- 长度锚点（Ant-only）：工具调用间 ≤25 词，最终回复 ≤100 词

### 有趣的元指令

- **`@[MODEL LAUNCH]`**：模型发布时需更新的标记
- **前沿模型**：当前为 `Claude Opus 4.6`
- **`isUndercover()`**：隐身模式，完全隐藏模型信息
- **动态边界标记**：精确控制哪些内容可以跨请求缓存

## 其他常量文件

### `tools.ts` — 工具白名单/黑名单

| 常量 | 内容 |
|------|------|
| `ALL_AGENT_DISALLOWED_TOOLS` | 所有代理禁用的工具 |
| `ASYNC_AGENT_ALLOWED_TOOLS` | 异步代理允许的工具（FileRead/Edit/Write, Shell, Grep/Glob 等） |
| `IN_PROCESS_TEAMMATE_ALLOWED_TOOLS` | 进程内 teammate 额外允许（TaskCreate, SendMessage 等） |
| `COORDINATOR_MODE_ALLOWED_TOOLS` | 协调器模式（Agent, TaskStop, SendMessage, SyntheticOutput） |

### `system.ts` — 系统常量

3 种系统提示前缀：
- `DEFAULT_PREFIX`："You are Claude Code, Anthropic's official CLI for Claude."
- `AGENT_SDK_CLAUDE_CODE_PRESET_PREFIX`：同上 + "running within the Claude Agent SDK."
- `AGENT_SDK_PREFIX`："You are a Claude agent..."

归因头（`getAttributionHeader`）包含 `cc_version`、`cc_entrypoint`、`cc_workload`，支持原生客户端证明。

### `apiLimits.ts` — API 限制

| 常量 | 值 | 说明 |
|------|----|------|
| 图片 base64 最大 | 5 MB | API 硬限 |
| 图片原始目标 | 3.75 MB | 编码前尺寸 |
| 图片最大维度 | 2000px | 客户端限制 |
| PDF 原始最大 | 20 MB | API 请求限 |
| PDF 最大页数 | 100 | API 硬限 |
| 每请求最大媒体项 | 100 | 超过报错 |

### `betas.ts` — Beta 功能头

| Header | 功能 |
|--------|------|
| `interleaved-thinking-2025-05-14` | 交错思考 |
| `context-1m-2025-08-07` | 1M 上下文 |
| `web-search-2025-03-05` | Web 搜索 |
| `task-budgets-2026-03-13` | 任务预算 |
| `fast-mode-2026-02-01` | 快速模式 |
| `token-efficient-tools-2026-03-28` | Token 高效工具 |
| `advisor-tool-2026-03-01` | 顾问工具 |

### `oauth.ts` — OAuth 配置

| 作用域组 | Scopes |
|----------|--------|
| Console | `org:create_api_key`, `user:profile` |
| Claude.ai | `user:profile`, `user:inference`, `user:sessions:claude_code`, `user:mcp_servers` |

### `toolLimits.ts` — 工具结果限制

- 单个工具结果：50K 字符
- Token 上限：100K
- 每消息：200K 字符

### 其他小文件

| 文件 | 内容 |
|------|------|
| `xml.ts` | XML 标签名定义（bash-input/stdout, task-notification, tick 等） |
| `outputStyles.ts` | 输出风格（Explanatory、Learning）+ 插件自定义 |
| `product.ts` | 产品 URL + Session ID 环境检测 |
| `cyberRiskInstruction.ts` | 安全风险指令（安全团队维护，禁止擅改） |
| `figures.ts` | UI 图标字符 |
| `spinnerVerbs.ts` | 加载动画动词 |
| `files.ts` | 文件类型常量 |
| `common.ts` | 日期工具函数 |

## 总结

`constants/` 是 Claude Code 的**声明式配置核心**。`prompts.ts` 是灵魂文件——53KB 的系统提示词定义了 Claude Code 的行为边界、工具使用规范和输出风格，精心设计的静态/动态分区优化了 prompt cache 命中率。
