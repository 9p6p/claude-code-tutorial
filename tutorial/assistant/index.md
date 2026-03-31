# `assistant/` — 远程会话历史

## 模块概述

`assistant/` 提供远程 CCR 会话的**历史事件分页加载**功能。

## 文件：`sessionHistory.ts`

### 分页 API

```typescript
export const HISTORY_PAGE_SIZE = 100

export async function fetchLatestEvents(
  ctx: HistoryAuthCtx, limit?: number
): Promise<HistoryPage | null>

export async function fetchOlderEvents(
  ctx: HistoryAuthCtx, beforeId: string, limit?: number
): Promise<HistoryPage | null>
```

使用**基于游标的分页**（before_id / anchor_to_latest），支持向前和向后翻页。

### 认证上下文

```typescript
export async function createHistoryAuthCtx(
  sessionId: string,
): Promise<HistoryAuthCtx> {
  const { accessToken, orgUUID } = await prepareApiRequest()
  return {
    baseUrl: `${BASE_API_URL}/v1/sessions/${sessionId}/events`,
    headers: { ...getOAuthHeaders(accessToken), ... },
  }
}
```

一次创建认证上下文，跨页面复用。

## 总结

一个简洁的 HTTP 客户端模块（88 行），实现了远程会话事件的分页加载。错误安全（catch 返回 null），15 秒超时。
