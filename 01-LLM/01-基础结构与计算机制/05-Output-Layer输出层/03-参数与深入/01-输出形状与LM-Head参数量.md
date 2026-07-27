---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer参数与深入概览|Output-Layer参数与深入概览]]"
previous: "[[00-Output-Layer参数与深入概览|Output-Layer参数与深入概览]]"
next: "[[02-Weight-Tying是什么|Weight-Tying是什么]]"
tags: [llm, lm-head, shape, parameters]
---

# 输出形状与 LM Head 参数量

> [!summary]
> LM Head 对每个 Token 位置执行 `hidden_size → vocab_size` 的投影，因此把主干形状的最后一维从 `hidden_size` 替换成 `vocab_size`。

## 先看形状变化

输入 Final Hidden States：

```text
[batch_size, sequence_length, hidden_size]
```

经过 Final Norm 后形状不变，再经过 LM Head：

```text
[batch_size, sequence_length, hidden_size]
→ [batch_size, sequence_length, vocab_size]
```

Batch 数量和 Token 位置数量没有被 LM Head 改变；改变的是每个位置最后一维所表达的空间。

## 一个教学形状

假设：

```text
batch_size     = 2
sequence_length = 3
hidden_size    = 4
vocab_size     = 10
```

那么：

```text
Final Hidden States：[2,3,4]
Logits：             [2,3,10]
```

含义是 2 条序列、每条 3 个位置、每个位置有 10 个词表候选分数。

## LM Head 权重形状

从概念上看，权重矩阵连接：

```text
hidden_size × vocab_size
```

不同框架保存矩阵时可能写成 `[vocab_size, hidden_size]` 或在计算中以转置形式使用。不要只凭数组朝向判断职责，先确认它执行的是 `hidden_size → vocab_size`。

## 参数量怎样估算

若不共享权重，且 LM Head 没有 Bias：

```text
参数量 = hidden_size × vocab_size
```

教学示例：

```text
hidden_size = 4
vocab_size  = 10

LM Head 参数量 = 4 × 10 = 40
```

如果实现还包含每个词表项一个 Bias，则再增加 `vocab_size` 个参数。许多现代 Causal LM 的输出投影不使用 Bias，但必须以具体实现为准。

## 为什么输出层可能很大

词表通常包含数万到十几万个 Token。即使 LM Head 只有一次线性投影，`hidden_size × vocab_size` 仍可能形成很大的矩阵。

这也是 [[02-Weight-Tying是什么|Weight Tying]] 有意义的原因之一：共享权重可以减少独立参数，但是否共享是模型设计选择，不是所有模型都一样。

## 理解检查

1. `[2,3,10]` 中的 10 表示什么？
2. LM Head 为什么通常不改变 `sequence_length`？
3. 为什么权重文件中的矩阵朝向可能不同，但功能仍相同？

下一篇：[[02-Weight-Tying是什么|Weight Tying 是什么]]。
