---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-训练目标与参数更新概览|训练目标与参数更新概览]]"
previous: "[[00-预测目标与Labels概览|预测目标与 Labels 概览]]"
next: "[[00-Backward与参数更新概览|Backward 与参数更新概览]]"
tags: [llm, forward, logits, loss, cross-entropy]
---

# Forward 与 Loss 概览

> [!summary]
> Forward 用当前参数产生预测，Loss 再把预测与 Labels 的偏差变成一个可优化的数值；两者都还没有直接修改参数。

## 主线

```text
input_ids
→ Embedding 与 Transformer Forward
→ 每个位置的 Hidden State
→ LM Head
→ Logits
→ 与 Labels 比较
→ 每个有效位置的 Loss
→ 汇总为当前 Batch Loss
→ 作为整体训练目标的一次局部估计
```

## 三个核心问题

1. [[01-Forward怎样产生Hidden-States与Logits|Forward 怎样产生 Hidden States 与 Logits]]
2. [[02-Loss与Cross-Entropy怎样衡量预测偏差|Loss 与 Cross-Entropy 怎样衡量预测偏差]]
3. [[03-从Token-Loss到整体训练目标|从 Token Loss 到整体训练目标]]

## 边界

- Logits 是 Vocabulary 原始分数，不是概率和 Gradient。
- Loss 衡量训练目标中的预测偏差，不等于现实世界的“整体错误程度”。
- Loss 下降通常是训练的重要信号，但不能单独证明模型在所有任务上更好。
