---
type: review
module: 1
status: complete
audience: non-specialist
parent: "[[完整Transformer-Block串联复习概览]]"
previous: "[[一次Block完整数据流]]"
next: "[[Transformer基础结构最终检查]]"
tags: [llm, transformer-block, tensor-shape, worked-example]
---

# 完整 Block 形状追踪

> [!summary]
> 使用一个极小形状追踪完整 Block，可以看到 Attention 的多 Head 组织和 FFN 的中间展开都只是内部变化，Residual 主干最终保持 `[batch_size, sequence_length, hidden_size]`。

> [!warning] 教学示例
> 下面只追踪形状，不计算真实权重和向量。所有尺寸均为人为设计，不属于真实模型。

## 教学配置

```text
batch_size = 1
sequence_length = 3
hidden_size = 4
num_attention_heads = 2
head_dim = 2
intermediate_size = 8
```

因为：

```text
2 个 Head × 每个 Head 2 维 = hidden_size 4
```

## Block 输入

```text
X：[1,3,4]
```

解释：

```text
1 个样本
3 个 Token 位置
每个位置 4 维 Hidden State
```

## Norm 1

Norm 对每个位置的 4 个数分别整理：

```text
[1,3,4]
→ Norm
→ [1,3,4]
```

形状不变。

## Q、K、V 与多个 Head

教学上先假设 Q、K、V 都产生相同总宽度：

```text
Q：[1,3,4]
K：[1,3,4]
V：[1,3,4]
```

组织成两个 Head：

```text
[1,3,4]
→ [1,2,3,2]

含义：
[batch, heads, positions, head_dim]
```

GQA、MQA 和 MLA 会改变 K/V 的具体 Head 或潜在表示组织；这里使用最简单 MHA 教学形状。

## Attention Score

每个 Head 中，3 个接收位置分别与 3 个可见候选位置比较：

```text
Score：[1,2,3,3]
```

最后两个 `3` 的含义不同：

```text
第一个 3 → Query 位置
第二个 3 → Key 位置
```

Causal Mask 会屏蔽不允许读取的未来位置，但不会因此删除张量中的 Token 位置。

## Value 汇总与 Head 合并

每个 Head 为三个位置各产生一个 2 维 Context：

```text
[1,2,3,2]
```

两个 Head 合并：

```text
[1,2,3,2]
→ [1,3,4]
```

经过 Output Projection 后仍是：

```text
Attention 输出：[1,3,4]
```

## 第一次 Residual

```text
X：               [1,3,4]
Attention 输出：  [1,3,4]
相加得到 A：      [1,3,4]
```

## Norm 2

```text
A：[1,3,4]
→ Norm
→ [1,3,4]
```

## FFN 展开与压回

```text
FFN 输入：       [1,3,4]
展开后：         [1,3,8]
Activation/Gate：[1,3,8]
压回后：         [1,3,4]
```

`intermediate_size=8` 只改变最后一个特征维度，不增加 Batch 或 Token 位置数。

## 第二次 Residual

```text
A：          [1,3,4]
FFN 输出：   [1,3,4]
Block 输出 Y：[1,3,4]
```

## 完整形状路线

```text
[1,3,4]  Block 输入
→ [1,3,4] Norm 1
→ [1,2,3,2] 多 Head Q/K/V 组织
→ [1,2,3,3] Attention Score / Weight
→ [1,2,3,2] 各 Head Context
→ [1,3,4] Head 合并与输出投影
→ [1,3,4] 第一次 Residual
→ [1,3,4] Norm 2
→ [1,3,8] FFN 中间表示
→ [1,3,4] FFN 输出
→ [1,3,4] 第二次 Residual / Block 输出
```

## 为什么这个例子重要

它同时说明：

- Attention 跨位置混合，但主干位置数通常不变；
- Head 拆分是内部组织，不是增加 Token；
- `intermediate_size` 是 FFN 内部宽度；
- Residual 要求分支最终回到主干形状；
- 下一个 Block 可以直接接收 `[1,3,4]`。

## 理解检查

1. Score 的 `[1,2,3,3]` 中两个 `3` 各代表什么？
2. 为什么 FFN 的 `[1,3,8]` 不会成为下一 Block 的主干形状？
3. 两次 Residual 为什么都能逐元素相加？

下一篇：[[Transformer基础结构最终检查|Transformer 基础结构最终检查]]。
