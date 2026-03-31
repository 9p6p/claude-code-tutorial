# `buddy/` — 虚拟伙伴系统

## 模块概述

一个彩蛋式的**虚拟宠物/伙伴系统**（6 个文件）——基于用户 ID 确定性生成 RPG 风格的虚拟角色。

## 确定性生成（不可作弊！）

```typescript
// Mulberry32 seeded PRNG + FNV-1a hash
function roll(userId: string): Roll {
  const rng = mulberry32(hashString(userId + SALT))
  return { species: pick(rng, SPECIES), rarity: rollRarity(rng), ... }
}
```

同一用户永远生成相同角色。**骨架**（外观）从 hash 派生，**灵魂**（名字/性格）由 AI 生成后存储。

## 稀有度系统

| 稀有度 | 权重 | 概率 |
|--------|------|------|
| Common | 60 | 60% |
| Uncommon | 25 | 25% |
| Rare | 10 | 10% |
| Epic | 4 | 4% |
| **Legendary** | 1 | **1%** |

## 18 种物种

duck · goose · blob · cat · dragon · octopus · owl · penguin · turtle · snail · ghost · axolotl · capybara · cactus · robot · rabbit · mushroom · chonk

## 文件清单

| 文件 | 说明 |
|------|------|
| `roll.ts` | 确定性随机生成器 |
| `prompt.ts` | AI 灵魂生成提示词 |
| `CompanionSprite.tsx` | ASCII 精灵渲染 |
| `useBuddyNotification.ts` | 触发通知（`/buddy` 命令） |
| `species.ts` | 物种定义和 ASCII art |

## 总结

`buddy/` 是工程严谨性与趣味性的完美结合——确定性 PRNG 保证公平，骨架/灵魂分离避免作弊，ASCII 精灵渲染为终端带来生命力。
