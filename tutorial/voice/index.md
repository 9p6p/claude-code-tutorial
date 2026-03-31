# `voice/` — 语音模式门控

## 模块概述

`voice/` 模块控制 Claude Code 的**语音模式**可用性——通过 Feature Flag 和认证检查决定语音功能是否启用。

## 文件：`voiceModeEnabled.ts`

### 三层门控

```typescript
// 1. 编译时：Feature Flag
feature('VOICE_MODE')

// 2. 运行时：GrowthBook kill-switch
function isVoiceGrowthBookEnabled(): boolean {
  return !getFeatureValue_CACHED_MAY_BE_STALE(
    'tengu_amber_quartz_disabled', false
  )
}

// 3. 运行时：认证检查
function hasVoiceAuth(): boolean {
  if (!isAnthropicAuthEnabled()) return false
  const tokens = getClaudeAIOAuthTokens()
  return Boolean(tokens?.accessToken)
}
```

语音模式**仅支持 Anthropic OAuth**，不支持 API Key、Bedrock、Vertex 或 Foundry。

### 完整检查

```typescript
export function isVoiceModeEnabled(): boolean {
  return hasVoiceAuth() && isVoiceGrowthBookEnabled()
}
```

## 总结

一个精简的功能门控模块（55 行），展示了三层防护设计：编译时裁剪 → 运行时 kill-switch → 认证验证。
