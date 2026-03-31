# `constants/` — 常量与配置中心

## 模块概述

`constants/` 包含 21 个文件，是全应用范围的常量定义。其中 `prompts.ts`（53KB）是最大文件，包含完整的系统提示词。

## 文件清单（按重要性）

| 文件 | 大小 | 说明 |
|------|------|------|
| `prompts.ts` | **53KB** | 系统提示词（最核心！） |
| `outputStyles.ts` | 9.7KB | 输出样式配置 |
| `oauth.ts` | 8.8KB | OAuth 配置 |
| `tools.ts` | 4.6KB | 工具相关常量 |
| `system.ts` | 3.8KB | 系统常量 |
| `apiLimits.ts` | 3.4KB | API 限制参数 |
| `xml.ts` | 3.3KB | XML 标签常量 |
| `files.ts` | 2.6KB | 文件相关常量 |
| `product.ts` | 2.5KB | 产品信息 |
| `betas.ts` | 2.2KB | Beta 功能标识 |
| 其余 11 个 | <2KB | 各种小常量 |

## 总结

`constants/` 是配置中心，`prompts.ts` 定义了 Claude Code 的"人格"——53KB 的系统提示是理解 Claude Code 行为的关键。
