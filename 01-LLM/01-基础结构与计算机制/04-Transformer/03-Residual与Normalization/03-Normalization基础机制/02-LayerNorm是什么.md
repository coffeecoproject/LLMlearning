---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Normalization基础机制概览|Normalization基础机制概览]]"
previous: "[[01-为什么需要Normalization|为什么需要Normalization]]"
next: "[[03-RMSNorm是什么|RMSNorm是什么]]"
tags: [llm, layernorm, normalization, mean, variance]
---

# LayerNorm 是什么

> [!summary]
> LayerNorm 对一个 Token 位置的 Hidden State 先减去该向量的均值，再按方差尺度缩放，最后使用可学习参数进行调节。

## 先看处理对象

假设主干形状是：

```text
[batch_size, sequence_length, hidden_size]
```

LayerNorm 通常对每个 Batch 样本中的每个 Token 位置分别处理，统计最后一个 `hidden_size` 维度：

```text
位置 1 的 [hidden_size] 向量 → 单独 LayerNorm
位置 2 的 [hidden_size] 向量 → 单独 LayerNorm
```

因此序列中不同位置不会因为 LayerNorm 而互相混合信息。跨位置交流仍是 Attention 的职责。

## 三个主要动作

### 1. 减去均值

先找出这个向量各维度的平均值，再从每一维减去它，使数值围绕零重新居中。

### 2. 按方差尺度缩放

再根据数值分散程度进行缩放，使整体尺度更统一。

### 3. 可学习调节

标准 LayerNorm 通常还有：

```text
gamma / weight → 每个维度的可学习 Scale
beta / bias    → 每个维度的可学习偏移
```

不同实现可能省略 Bias，因此判断真实模型要查看代码和配置。

## 可选技术轮廓

不要求推导，只认识结构：

```text
均值 μ = 向量各维度平均
方差 σ² = 各维度围绕均值的平方偏差平均

输出
= (x - μ) / sqrt(σ² + eps)
  × gamma
  + beta
```

`eps` 是一个很小的正数，用于避免分母过小造成数值问题。它是配置中的数值稳定参数，不是模型能力分数。

## 参数量怎样理解

如果：

```text
hidden_size = H
```

带 Scale 和 Bias 的 LayerNorm 通常有：

```text
gamma：[H]
beta： [H]
```

大约 `2H` 个可学习参数。与 Attention 和 FFN 的大型矩阵相比通常很少，但结构作用很重要。

## 它没有做什么

- 不改变 Token 数量；
- 通常不改变 `hidden_size`；
- 不在 Token 位置之间交换内容；
- 不把向量永久保存成模型知识；
- 不保证经过可学习 Scale/Bias 后最终均值仍严格为零。

## 与 Layer 这个词的关系

LayerNorm 是算法名称，不表示它只在“最后一层”执行，也不表示一次 LayerNorm 等于一个 Transformer Layer。一个 Block 内可以有多个 Norm 位置。

## 常见误解

- LayerNorm 通常不是跨 Batch 统计。
- “减均值”不表示删除语义信息。
- `eps` 不是可忽略的模型层，而是数值稳定项。
- LayerNorm 自身不负责上下文混合。

## 理解检查

1. LayerNorm 通常对哪个维度计算均值和方差？
2. `gamma` 与 `beta` 分别起什么作用？
3. LayerNorm 为什么不会代替 Attention？

下一篇：[[03-RMSNorm是什么|RMSNorm 是什么]]。
