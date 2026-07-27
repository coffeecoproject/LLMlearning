---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[Block堆叠基础机制概览]]"
previous: "[[Block与Layer是什么关系]]"
next: "[[为什么要堆叠多个Block]]"
tags: [llm, hidden-state, transformer-block, residual-stream]
---

# Hidden State 怎样逐层流动

> [!summary]
> 每个 Block 接收整条序列当前的 Hidden States，产生一次更新后的 Hidden States；该输出直接成为下一 Block 的输入。

## 从初始表示开始

Tokenizer 产生 Token ID 后，Embedding 为每个位置取出初始向量。结合对应位置机制后，可以把进入 Transformer 主体的整组表示记作：

```text
H⁰
```

它表示“第 0 层状态”，不是零向量。

## 一层接一层

```text
H⁰ → Block 1 → H¹
H¹ → Block 2 → H²
H² → Block 3 → H³
```

`H¹` 同时具有两个身份：

```text
Block 1 的输出
= Block 2 的输入
```

模型不会在两层之间把 Hidden State 解码回文字，也不会重新经过 Tokenizer 或 Embedding。

## 放大一个 Block

以常见串行 Pre-Norm Block 为例：

```text
Hˡ
→ Norm
→ Attention
→ 第一次 Residual，得到中间状态 Aˡ
→ Norm
→ FFN
→ 第二次 Residual
→ Hˡ⁺¹
```

可选技术记号：

```text
Aˡ   = Hˡ + Attention(Norm(Hˡ))
Hˡ⁺¹ = Aˡ + FFN(Norm(Aˡ))
```

公式只是在记录两次“主干 + 子层变化”，不要求做矩阵推导。Post-Norm、并行 Attention/FFN 等架构会改变具体排列。

## Hidden State 是一个还是很多个

日常说“某个 Token 的 Hidden State”，通常指一个位置的向量：

```text
[hidden_size]
```

说“这一层的 Hidden States”，通常指所有位置组成的张量：

```text
[sequence_length, hidden_size]
```

若包含 Batch，则常写成：

```text
[batch_size, sequence_length, hidden_size]
```

三种说法观察的是同一数据流的不同范围。

## 同一 Token 为什么会逐层变化

例如：

```text
我购买了苹果公司的股票
```

“苹果”位置初始表示主要来自对应 Token ID。经过 Attention 后，它能吸收“公司”“股票”等可见位置的影响；经过 FFN 和 Residual 后形成更新表示。下一层再在这个更新结果上继续计算。

因此：

```text
同一位置
→ 每层仍对应同一个序列位置
→ 但 Hidden State 数值不断变化
```

## 它不是参数

```text
模型参数
→ 训练后保存在模型权重中

Hidden State
→ 根据当前输入临时计算出来
```

换一个 Prompt，即使模型参数不变，也会产生不同 Hidden States。

## 常见误解

- Hidden State 不会在层间变回 Token ID。
- 上一层输出不会绕过下一层直接变成最终答案。
- Hidden State 不是模型刚刚永久写入的新知识。
- `H³` 的上标表示层次，不表示三次方。

## 理解检查

1. `H¹` 对 Block 1 和 Block 2 分别是什么？
2. 为什么普通运行时参数不变，Hidden States 仍会随 Prompt 改变？
3. 单个位置向量与整条序列 Hidden States 的形状有什么关系？

下一篇：[[为什么要堆叠多个Block|为什么要堆叠多个 Block]]。
