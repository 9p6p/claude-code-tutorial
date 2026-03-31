# `commands/` — 斜杠命令实现大全

## 模块概述

`commands/` 包含 Claude Code 所有 **67+ 斜杠命令** 的实现（293 个文件）。用户在 REPL 中输入 `/xxx` 触发对应命令。

## 命令类型

| 类型 | 说明 | 数量 |
|------|------|------|
| `prompt` | 生成提示词发送给模型 | ~8 |
| `local` | 本地执行，返回纯文本 | ~14 |
| `local-jsx` | 本地执行，渲染 React UI | ~45 |

## 命令分类索引

### 会话管理
`/clear` · `/compact` · `/resume` · `/session` · `/rename` · `/branch` · `/rewind` · `/copy` · `/export` · `/exit`

### 代码操作
`/commit` · `/commit-push-pr` · `/diff` · `/review` · `/ultrareview` · `/security-review` · `/init` · `/plan` · `/ultraplan`

### 配置管理
`/config` · `/permissions` · `/theme` · `/vim` · `/keybindings` · `/model` · `/fast` · `/effort` · `/advisor` · `/color` · `/memory` · `/sandbox` · `/hooks`

### MCP/插件
`/mcp` · `/plugin` · `/skills` · `/reload-plugins` · `/agents`

### 信息查看
`/help` · `/cost` · `/usage` · `/status` · `/doctor` · `/stats` · `/release-notes` · `/insights` · `/context`

### 认证/账户
`/login` · `/logout` · `/upgrade` · `/passes`

### 集成
`/install-github-app` · `/install-slack-app` · `/chrome` · `/ide` · `/remote-control` · `/web-setup`

### 特色功能
`/mobile` · `/desktop` · `/voice` · `/stickers` · `/tasks` · `/btw` · `/feedback` · `/think-back` · `/buddy`

## 架构亮点

### 懒加载
```typescript
const compact = {
  type: 'local',
  name: 'compact',
  description: 'Clear conversation history but keep a summary',
  load: () => import('./compact.js'),  // 懒加载！
} satisfies Command
```

所有命令的 `index.ts` 仅包含元数据，实际实现通过 `load()` 懒加载。

### Feature Gate
```typescript
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

### 插件迁移模式
```typescript
export default createMovedToPluginCommand({
  name: 'pr-comments',
  pluginName: 'pr-comments',
  getPromptWhileMarketplaceIsPrivate: async (args) => [...]
})
```

已迁移到插件的命令优雅降级——内部用户看安装提示，外部用户继续用内联 prompt。

## 总结

67+ 命令覆盖了从代码操作、配置管理到社交集成的所有场景。懒加载 + Feature Gate 确保启动性能，命令类型系统（prompt/local/local-jsx）清晰区分了与 AI 的交互方式。
