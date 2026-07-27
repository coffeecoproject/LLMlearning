---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN基础机制概览|FFN基础机制概览]]"
previous: "[[02-FFN在Transformer-Block中的位置与关联|FFN在Transformer-Block中的位置与关联]]"
next: "[[04-FFN小数字完整计算|FFN小数字完整计算]]"
tags: [llm, ffn, mlp, dense, activation, projection]
---

# FFN 核心机制与关键名词

> [!summary]
> 基础 Dense FFN 对每个位置执行同一套“升维投影 → 非线性处理 → 降维投影”，把当前 Hidden State 转换成可加入 Residual 的变化向量。

## 先看完整机制，再解释名词

经典基础路线：

```text
输入 Hidden State
→ Up Projection
→ Activation Function
→ Down Projection
→ FFN 输出
```

形状路线：

```text
hidden_size
→ intermediate_size
→ intermediate_size
→ hidden_size
```

现代门控 FFN 会增加 Gate 分支，但第一遍先理解这条基线。

## FFN 与 MLP

```text
FFN = Feed-Forward Network
MLP = Multi-Layer Perceptron
```

在大模型代码中经常看到 `mlp`，结构图中写 FFN。多数情况下，它们指 Transformer Block 中承担同一职责的子层，不是两个连续模块。

`FNN` 是一种更宽泛的 Feedforward Neural Network 缩写；本学习库统一采用 Transformer 资料更常见的 `FFN`。

## Dense FFN

`Dense` 表示进入这一层的每个 Token 位置都会使用同一整套主要 FFN 参数：

```text
位置 1 → 同一套 FFN
位置 2 → 同一套 FFN
位置 3 → 同一套 FFN
```

不同位置输入不同，因此输出不同。同一层共享参数，不表示不同 Transformer Block 也共享同一套参数。

Dense 与 MoE 的区别先只记：

```text
Dense：每个 Token 走同一套完整 FFN
MoE：每个 Token 从多套 Expert FFN 中选择少数路径
```

## Up Projection 与 intermediate_size

Up Projection 把每个位置从主干宽度映射到更宽的中间空间：

```text
hidden_size = 4
intermediate_size = 8

[4] → [8]
```

升维没有凭空增加外部事实，只是把输入重新组合为更多中间数值，为后续非线性或门控提供更宽的计算空间。

## 为什么需要非线性

如果只有连续两个纯线性投影：

```text
x → 线性 A → 线性 B
```

它们整体仍可以合并成另一个线性变换。中间加入 Activation Function，才能形成不能简单压缩成单个线性矩阵的输入依赖变化。

为了建立直觉，可以先认识 ReLU：

```text
ReLU(-2) = 0
ReLU(3)  = 3
```

现代 LLM 常使用 SiLU、GELU 或 SwiGLU 等结构；公式和差异放入扩展阅读。

## Down Projection

Down Projection 把中间宽度带回主干 `hidden_size`：

```text
[8] → [4]
```

这一步不删除 Token 位置。它改变的是每个位置的特征宽度，并让 FFN 输出可以与 Residual 主干相加。

## Position-wise：逐位置处理

假设序列有三个位置：

```text
FFN(h₁)
FFN(h₂)
FFN(h₃)
```

这个 FFN 子层不会让 `h₁` 直接读取 `h₂`。但是 `h₁`、`h₂`、`h₃` 已经经过 Attention，所以它们可能包含其他位置的影响。

程序可以并行计算所有位置，但并行执行不等于位置间信息混合。

## 名词速查

| 名词 | 最短解释 |
|---|---|
| FFN / MLP | Block 中逐位置处理 Hidden State 的子层 |
| Dense | 每个位置使用这一层完整主要参数路径 |
| `hidden_size` | Transformer 主干向量宽度 |
| `intermediate_size` | FFN 内部暂时使用的中间宽度 |
| Up Projection | `hidden_size → intermediate_size` |
| Activation | 加入非线性变化 |
| Down Projection | `intermediate_size → hidden_size` |
| Gate | 根据输入调节中间特征通过强度，属于现代扩展 |

## 常见误解

- `intermediate_size` 不是 Token 数量。
- 升维不是创建更多 Token。
- 激活函数不是从词表激活一个词。
- FFN 不计算 Causal Mask。
- Dense 不表示所有模型都没有并行部署。

## 理解检查

1. 为什么两个纯线性投影之间需要非线性？
2. `hidden_size` 与 `intermediate_size` 分别位于哪里？
3. Down Projection 为什么与 Residual 接口有关？
4. “逐位置”为什么不等于“没有上下文”？

下一篇：[[04-FFN小数字完整计算|FFN 小数字完整计算]]。如果暂时不想看数字，可以返回 [[00-FFN概览|FFN概览]] 继续整体学习。
