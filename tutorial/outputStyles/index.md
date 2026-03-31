# `outputStyles/` — 输出样式加载

## 模块概述

`outputStyles/` 从项目和用户目录加载**自定义输出样式**——用 Markdown 文件定义 AI 的回复风格。

## 文件：`loadOutputStylesDir.ts`

### 加载路径

```
项目级：.claude/output-styles/*.md  （优先级高）
用户级：~/.claude/output-styles/*.md（被项目级覆盖）
```

### 样式格式

每个 Markdown 文件的 frontmatter 定义元数据：

```yaml
---
name: concise
description: 简洁明了的回复风格
keep-coding-instructions: true
---

请用简短、直接的语言回复。不要使用废话或填充词。
```

文件名（去掉 .md）作为样式名称，Markdown 内容作为 AI 的风格提示。

### 缓存

```typescript
export const getOutputStyleDirStyles = memoize(
  async (cwd: string): Promise<OutputStyleConfig[]> => {
    // 加载并解析 Markdown 文件
  }
)
```

使用 `lodash.memoize` 缓存加载结果。

## 总结

一个简洁的配置加载器，让用户通过 Markdown 文件自定义 AI 的回复风格。
