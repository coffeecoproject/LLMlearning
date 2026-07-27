---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[Multi-Head-Attention概览]]"
previous: "[[为什么需要多个Head]]"
next: "[[每个Head怎样独立产生Context]]"
tags: [llm, qkv, reshape, head-dimension, tensor-shape]
---

# QKV 怎样组织成多个 Head

> [!summary]
> 模型通常先用可学习投影从输入 Hidden States 产生包含所有 Head 的 Q、K、V，再通过形状重组把 Head 维度显式分开；它不是把原始 Hidden State 随手切成几段就结束。

## 先看可观察的变化

下面使用教学形状：

```text
sequence_length = 3
hidden_size = 8
num_heads = 2
head_dim = 4
```

输入表示：

```text
H：[3, 8]
```

产生所有 Head 的 Q、K、V：

```text
Q_all：[3, 8]
K_all：[3, 8]
V_all：[3, 8]
```

随后把最后的 8 维重新组织成 `2 × 4`：

```text
Q：[2, 3, 4]
K：[2, 3, 4]
V：[2, 3, 4]
     ↑  ↑  ↑
   Head 位置 Head维度
```

这样每个 Head 都得到形状 `[3,4]` 的 Q、K、V 序列。

## 为什么必须先投影

输入 Hidden State 是模型当前的通用表示。Q、K、V 是为了 Attention 匹配和信息传递而产生的角色表示：

```text
H × W_Q → Q_all
H × W_K → K_all
H × W_V → V_all
```

这些投影包含可学习参数。正是不同参数分区，让不同 Head 能从同一 H 中提取不同的 Query、Key 和 Value 表示。

如果只是把原始 H 按位置切成几段，却没有可学习的 Q/K/V 投影，就丢失了“同一输入转成三种角色表示”的核心机制。

## “每个 Head 各有矩阵”和“一次大投影”矛盾吗

不矛盾。理解上可以写成：

```text
Head 1：W_Q1、W_K1、W_V1
Head 2：W_Q2、W_K2、W_V2
```

工程实现中也可以把它们合并成更大的 `W_Q、W_K、W_V`，一次计算后再 Reshape。还可以进一步把 QKV 融合到一次大矩阵运算中。

这些写法改变参数存放和计算组织，不改变我们当前需要掌握的逻辑：每个 Head 最终拥有自己的 Q/K/V 子表示。

## Reshape 做了什么

Reshape 主要改变张量维度的解释方式：

```text
[sequence_length, num_heads × head_dim]
→ [num_heads, sequence_length, head_dim]
```

它本身不学习语义，也不新增参数。学习发生在产生 Q、K、V 的投影参数中；Reshape 只是让后续程序能按 Head 分别计算。

## 简单乘法关系

在这个标准 MHA 教学例子中：

```text
num_heads × head_dim = 2 × 4 = 8
```

因此拼接所有 Head 后能得到 8 维表示。真实模型要以其公开配置和实现为准；尤其进入 GQA、MQA 和 MLA 后，Query Head 与 K/V Head 的数量和表示组织可能不同。

## 省略 Batch 后意味着什么

本篇为了聚焦机制使用 `[S,H]`。如果一次处理多个序列，会多一个 Batch 维度：

```text
输入：[batch_size, sequence_length, hidden_size]
Q：  [batch_size, num_heads, sequence_length, head_dim]
```

这没有改变单条序列内部多 Head Attention 的逻辑。Batch 怎样补齐和调度属于运行或训练组织，不在这里展开。

## 常见误解

- **“多 Head 就是直接平均切开原始 Embedding。”** 通常先经过可学习的 Q/K/V 投影，再组织 Head 维度。
- **“Reshape 会学习不同语义。”** Reshape 没有可学习参数。
- **“一次大投影意味着所有 Head 参数相同。”** 大矩阵内部可以包含对应不同 Head 的参数分区。
- **“Q、K、V 的形状永远完全相同。”** 标准 MHA 可先这样理解，但 GQA、MQA、MLA 会改变部分组织方式。

## 理解检查

1. 为什么不能把多 Head 简化为“直接切开原始 Hidden State”？
2. 从 `[3,8]` 怎样组织成两个 `[3,4]` 的 Head？
3. Projection 和 Reshape 哪一步包含可学习参数？
4. 一次大 Q 投影为什么仍能产生多个不同 Head？

下一篇：[[每个Head怎样独立产生Context]]。
