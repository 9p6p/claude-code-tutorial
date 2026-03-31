# `buddy/` — 虚拟伙伴系统

## 模块概述

`buddy/` 实现了一个彩蛋式的**虚拟宠物/伙伴系统**（6 个文件）——基于用户 ID 确定性生成一个 RPG 风格的虚拟角色，拥有 18 种物种、5 档稀有度、RPG 属性值和 ASCII 精灵动画。

## 关键设计

### 确定性生成（不可作弊！）

```typescript
// Mulberry32 seeded PRNG + FNV-1a hash
function roll(userId: string): Roll {
  const rng = mulberry32(hashString(userId + SALT))
  return { species: pick(rng, SPECIES), rarity: rollRarity(rng), ... }
}
```

同一用户永远生成相同角色。骨架（外观）从 hash 派生，灵魂（名字/性格）由 AI 生成后存储。

### 稀有度系统

| 稀有度 | 权重 | 概率 |
|--------|------|------|
| Common | 60 | 60% |
| Uncommon | 25 | 25% |
| Rare | 10 | 10% |
| Epic | 4 | 4% |
| Legendary | 1 | 1% |

### 18 种物种

duck, goose, blob, cat, dragon, octopus, owl, penguin, turtle, snail, ghost, axolotl, capybara, cactus, robot, rabbit, mushroom, chonk

## 总结

`buddy/` 是工程严谨性与趣味性的完美结合——确定性 PRNG 保证公平性，骨架/灵魂分离避免作弊，ASCII 精灵渲染为终端带来生命力。
