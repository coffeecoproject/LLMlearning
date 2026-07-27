---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[FFN参数与深入概览]]"
previous: "[[FFN参数与深入概览]]"
next: "[[FFN参数量与规模]]"
tags: [llm, ffn, tensor-shape, parameters]
---

# FFN 输入、输出、形状与参数

> [!summary]
> FFN 输入和最终输出都保持 Transformer 主干的 `hidden_size`；内部投影参数负责在 `hidden_size` 与更宽的 `intermediate_size` 之间转换。

## FFN 输入是什么

在常见 Pre-Norm Block 中：

```text
h = Attention 后第一次 Residual 的结果
FFN 分支输入 = Norm(h)
```

它是上下文化 Hidden State，不是：

- 原始文字；
- Token ID；
- Attention Weight；
- 词表概率。

## 单个位置的形状

设：

```text
hidden_size = H
intermediate_size = I
```

经典 FFN：

```text
[H] → Up Projection → [I]
[I] → Activation    → [I]
[I] → Down Projection → [H]
```

输入输出都是 `[H]`，中间暂时使用 `[I]`。

## 整条序列和 Batch 的形状

如果：

```text
batch_size = 2
sequence_length = 3
hidden_size = 4
intermediate_size = 8
```

形状路线：

```text
输入：      [2,3,4]
升维后：    [2,3,8]
激活后：    [2,3,8]
降维后：    [2,3,4]
Residual：  [2,3,4] + [2,3,4]
```

Batch 数和 Token 位置数没有被 FFN 改变，主要变化发生在最后一个特征维度。

## 哪些东西是参数

经典两投影 FFN：

```text
W_up：   H → I
W_down： I → H
```

某些模型还包含 Bias。现代门控 FFN通常有：

```text
W_gate： H → I
W_up：   H → I
W_down： I → H
```

这些矩阵中的数值是可学习参数。Activation Function 通常是固定数学函数，不拥有同等规模的参数矩阵。

## 矩阵形状为什么有两种写法

教材可能把 `H→I` 的矩阵写成 `[H,I]`；PyTorch 权重存储可能显示 `[I,H]`。这是向量放在左边还是右边、存储是否转置造成的约定差异。

第一遍只看映射方向：

```text
H → I
I → H
```

## 参数在训练和运行时怎样不同

> [!info] LLM 训练阶段
> Loss 经过反向传播产生梯度，更新 `W_up`、`W_down` 和可能存在的 `W_gate` 等参数。

> [!info] LLM 运行阶段
> 参数固定读取，当前 Hidden State 经过这些矩阵计算输出。运行一次不会自动把新事实永久写入 FFN 参数。

前向计算本身两阶段都会执行，区别在于之后是否更新参数。

## 为什么 FFN 输出必须回到 H

Residual 需要逐元素相加：

```text
主干 [H] + FFN输出 [H]
```

如果 FFN 输出停在 `[I]`，通常不能直接与 `[H]` 主干相加。因此 Down Projection 也承担恢复 Block 接口宽度的职责。

## 理解检查

1. `[2,3,4]` 中三个维度分别代表什么？
2. FFN 为什么通常只改变最后一个维度？
3. 哪些矩阵是可学习参数？
4. 训练和运行都会执行 FFN，最关键差别是什么？
5. Down Projection 为什么不能随意省略？

下一篇：[[FFN参数量与规模|FFN 参数量与规模]]。
