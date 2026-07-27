---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer基础机制概览|Output-Layer基础机制概览]]"
previous: "[[00-Output-Layer基础机制概览|Output-Layer基础机制概览]]"
next: "[[02-LM-Head是什么|LM-Head是什么]]"
tags: [llm, final-hidden-state, final-norm, output-layer]
---

# Final Hidden State 怎样进入输出层

> [!summary]
> 最后一个 Transformer Block 输出的仍是每个 Token 位置的 Hidden State；常见 Decoder-only 模型会先经过一次 Final Norm，再把每个位置分别交给 LM Head。

## 上游交来的是什么

假设一批输入的主干形状是：

```text
[batch_size, sequence_length, hidden_size]
```

经过所有 Transformer Block 后，形状通常仍保持不变。变化的是每个位置向量中的数值与上下文信息，而不是 Token 位置数量或主干宽度。

例如教学形状：

```text
[1, 4, 8]
```

表示：

- 1 条序列；
- 4 个 Token 位置；
- 每个位置有 8 维最终表示。

## 为什么还有 Final Norm

在许多 Pre-Norm 架构中，每个 Block 会在 Attention 或 FFN 前进行 Norm，但 Residual 主干在最后一个 Block 之后还会留下最终状态。因此模型常在进入 LM Head 前再做一次 Final Norm：

```text
最后一个 Block 的输出
→ Final Norm
→ LM Head
```

Final Norm 的作用仍是整理数值尺度，不是增加上下文、不改变词表、也不选择 Token。不同架构细节可能不同，不能把“所有模型必然完全相同”当成结论。

## 是只处理最后一个位置吗

模型前向计算通常可以为所有输入位置形成输出表示：

```text
位置 1 Final Hidden State → LM Head → 位置 1 的词表 Logits
位置 2 Final Hidden State → LM Head → 位置 2 的词表 Logits
位置 3 Final Hidden State → LM Head → 位置 3 的词表 Logits
……
```

“只读取最后一个有效位置”主要是普通自回归运行在选择下一个 Token 时的使用方式，不表示 Output Layer 的结构只能处理一个位置。

## 为什么叫 Final Hidden State

这里的 `Final` 表示它已经通过最后一个 Transformer Block；`Hidden` 表示它仍是模型内部连续向量，不是人类可直接阅读的答案；`State` 表示当前上下文下该位置的内部状态。

## 常见误解

- Final Hidden State 不是最后一个 Token 的 Token ID。
- Final Hidden States 可以包含所有位置，不只最后一个位置。
- Final Norm 不是 Softmax，两者处理对象和目的完全不同。
- Output Layer 不会回到原文中搜索答案。

## 理解检查

1. `[1,4,8]` 中的 4 和 8 分别表示什么？
2. Final Norm 为什么不会把 Hidden State 变成概率？
3. “运行时读取最后位置”为什么不等于“模型只能为最后位置计算 Logits”？

下一篇：[[02-LM-Head是什么|LM Head 是什么]]。
