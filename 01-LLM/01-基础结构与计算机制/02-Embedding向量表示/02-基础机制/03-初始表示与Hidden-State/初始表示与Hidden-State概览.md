---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Embedding基础机制概览]]"
tags: [llm, embedding, hidden-state, contextual-representation]
---

# 初始表示与 Hidden State

> [!summary]
> Embedding Lookup 给出进入模型前的初始表示；经过 Transformer 层后，每个位置的表示不断吸收上下文并变成 Hidden State。

## 子结构

1. [[初始Token-Embedding|初始 Token Embedding]] ✓
2. [[Hidden-State是什么|Hidden State 是什么]] ✓
3. [[同一Token在不同上下文中的变化|同一 Token 在不同上下文中的变化]] ✓
4. [[表示在层与层之间怎样传递|表示在层与层之间怎样传递]] ✓

## 必须分清

```text
同一个 Token ID
→ 初始 Token Embedding 通常相同
→ 放进不同上下文并经过模型层
→ 对应位置的 Hidden State 可以不同
```

## 边界

本节建立概念和因果关系，不解释 Attention、FFN、Residual Connection 怎样具体更新 Hidden State；这些属于后续专题。

## 完成标准

学完后应能解释：

1. Token Embedding 为什么只是起点；
2. Hidden State 中的 `hidden` 为什么不等于“不可观察的秘密”；
3. 同一 Token 为什么能在不同上下文中形成不同表示；
4. 为什么层数增加时张量形状通常不变，但其中数值和信息会变化。
