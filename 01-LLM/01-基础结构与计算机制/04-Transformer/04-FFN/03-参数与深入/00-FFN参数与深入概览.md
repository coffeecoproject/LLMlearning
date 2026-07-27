---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN概览|FFN概览]]"
previous: "[[04-FFN小数字完整计算|FFN小数字完整计算]]"
next: "[[01-FFN输入输出形状与参数|FFN输入输出形状与参数]]"
tags: [llm, ffn, tensor-shape, parameters, optional]
---

# FFN 参数与深入

> [!summary]
> 这一层把 FFN 的直觉翻译成真实模型配置中常见的形状、投影矩阵和参数规模；它不是理解整体框架的前置条件。

## 阅读前提

先能口述这条基础路线即可：

```text
Attention 后的 Hidden State
→ FFN 分别加工各位置
→ 输出同宽度变化
→ Residual 加回主干
```

## 阅读顺序

1. [[01-FFN输入输出形状与参数|FFN 输入、输出、形状与参数]]
   - `hidden_size` 与 `intermediate_size`；
   - 单个位置、序列和 Batch 的形状；
   - `W_up`、`W_gate`、`W_down` 哪些是参数。

2. [[02-FFN参数量与规模|FFN 参数量与规模]]
   - 参数量为什么会很大；
   - 用小数字理解参数从哪里来；
   - 用 Qwen3-8B 配置做教学估算。

## 数学边界

这里的计算只用于解释：

```text
结构宽度怎样决定矩阵大小
矩阵大小怎样影响参数数量
```

不展开矩阵推导、梯度计算和优化算法，也不要求背公式。

下一篇：[[01-FFN输入输出形状与参数|FFN 输入、输出、形状与参数]]。
