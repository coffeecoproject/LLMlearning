---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN基础机制概览|FFN基础机制概览]]"
previous: "[[01-FFN的直白理解|FFN的直白理解]]"
next: "[[03-FFN核心机制与关键名词|FFN核心机制与关键名词]]"
tags: [llm, ffn, attention, residual, output-layer]
---

# FFN 在 Transformer Block 中的位置与关联

> [!summary]
> 在常见串行 Transformer Block 中，FFN 位于 Attention 更新之后、第二次 Residual 相加之前；它接收上下文化 Hidden State，输出同宽度变化。

## 先看完整位置

使用常见 Pre-Norm 串行结构：

```text
Block 输入 x
→ Norm
→ Attention
→ 第一次 Residual 相加，得到 h
→ Norm
→ FFN
→ 第二次 Residual 相加，得到 y
→ y 进入下一个 Block
```

因此 FFN：

- 在 Transformer Block 内部；
- 不在 Attention 的 QKV 计算内部；
- 不在所有 Transformer Block 之后；
- 通常每个 Block 都有一套；
- 输出仍是 Hidden State，不是词表概率。

Post-Norm、并行 Attention/FFN 等架构会改变具体顺序，但不改变 FFN 作为 Block 内特征变换子层的基本职责。

## 上一步给 FFN 什么

Attention 让每个位置从允许读取的其他位置汇集信息。经过第一次 Residual 后，得到中间状态 `h`：

```text
h
→ 已经包含当前位置原有主干
→ 也包含本层 Attention 带来的上下文变化
```

在 Pre-Norm 结构中，FFN 分支通常接收 `Norm(h)`；Residual 主干仍保留 `h`，等待与 FFN 输出相加。

## FFN 给下一步什么

```text
FFN(Norm(h))
→ 当前层 FFN 产生的变化

y = h + FFN(Norm(h))
→ Block 输出
```

FFN 必须把内部中间表示压回 `hidden_size`，才能与 `h` 逐元素相加。

## Attention 与 FFN 的分工

| 问题 | Attention | FFN |
|---|---|---|
| 是否跨 Token 位置混合信息 | 是 | 当前子层内部通常否 |
| 是否计算 Q、K、V | 是 | 否 |
| 是否产生位置间 Weight | 是 | 否 |
| 主要职责 | 把上下文信息送到合适位置 | 加工每个位置当前表示 |
| 输出是否回到 Hidden State 主干 | 是 | 是 |

最短记忆：

```text
Attention：交流
FFN：加工
Residual：把变化加回主干
```

## “逐位置”为什么不等于没有上下文

FFN 本身不重新读取其他位置，但它的输入已经经过 Attention：

```text
其他 Token 的影响
→ Attention
→ 当前位置 Hidden State
→ FFN
```

所以 FFN 做的是逐位置计算，处理的却是上下文化表示。

## 与 Output Layer 的边界

FFN 外部形状通常仍是：

```text
[batch_size, sequence_length, hidden_size]
```

Output Layer 才会把 Final Hidden States 投影成：

```text
[batch_size, sequence_length, vocab_size]
```

虽然两者都有矩阵投影，但位置、形状和职责都不同。

## 与模型知识的关系先放在哪里

Attention 更直接组织当前输入中的 Token 关系；FFN 的训练参数会参与参数化模式和事实关联的调用。但这是扩展理解，不是把 FFN 定义成外部知识库。第一遍先记住数据流，研究边界见 [[01-FFN与模型知识的关系|FFN与模型知识的关系]]。

## 阶段标注

> [!info] 两阶段共同
> Attention、FFN 和两次 Residual 的前向主线在训练和普通运行时都会执行。训练阶段更新参数，普通运行阶段读取固定参数。

## 理解检查

1. FFN 在常见 Block 的哪两步之间？
2. 为什么 FFN 不跨位置计算，却仍可能利用上下文？
3. FFN 与 Output Layer 的输出形状有什么不同？
4. 第二次 Residual 的主干是最初的 `x`，还是第一次更新后的 `h`？

下一篇：[[03-FFN核心机制与关键名词|FFN 核心机制与关键名词]]。
