---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Block堆叠与Hidden-State流动概览|Block堆叠与Hidden-State流动概览]]"
previous: "[[00-Block堆叠与Hidden-State流动概览|Block堆叠与Hidden-State流动概览]]"
next: "[[00-Block堆叠基础机制概览|Block堆叠基础机制概览]]"
tags: [llm, transformer-block, hidden-state, framework, beginner]
---

# Block 堆叠一页看懂

> [!summary]
> Transformer 不是只执行一次 Attention 和 FFN，而是把许多结构相似、参数通常不同的 Block 顺序连接，让 Hidden State 被一层接一层地更新。

## 它位于哪里

```text
Tokenizer → Token ID
→ Embedding 与位置影响
→ 初始 Hidden States
→ Block 1
→ Block 2
→ ……更多 Block
→ Final Hidden States
→ Output Layer
```

## 一个 Block 做什么

一个常见 Block 内部组合：

```text
Attention
→ 在 Token 位置之间交流信息

FFN
→ 分别加工每个位置当前表示

Residual
→ 保留主干并加入子层变化

Normalization
→ 调整数值尺度
```

经过一个 Block，相当于完成一轮上下文交流和特征加工。

## 为什么要堆叠多个 Block

一轮更新不一定能组合所有复杂关系。下一层接收的已经不是最初 Embedding，而是上一层处理后的表示：

```text
第 1 层：在初始表示上更新
第 2 层：在第 1 层结果上继续更新
第 3 层：再组合前两层已经形成的信息
```

多层堆叠使模型可以反复读取、加工和重组上下文，但不能简单规定“某一层只负责语法、某一层只负责知识”。

## Hidden State 怎样流动

```text
Block 1 输出
= Block 2 输入

Block 2 输出
= Block 3 输入
```

Hidden State 是当前输入在某一层的临时数值表示。它不是新 Token，也不是模型刚刚永久学会的参数。

## 为什么形状通常不变

假设一条序列有 3 个 Token，每个位置主干宽度是 4：

```text
进入 Block：[3,4]
离开 Block：[3,4]
```

形状相同，是为了让 Residual 相加和下一层连接保持一致；内部数值和携带的上下文信息已经变化。

FFN 的 `intermediate_size` 只是 Block 内部临时展开的宽度：

```text
主干 hidden_size
→ FFN intermediate_size
→ 回到 hidden_size
→ 才离开 Block
```

它不会随着 Block 层数不断累加。

## 最后一层去哪里

```text
最后一个 Block 的输出
→ Final Hidden States
→ 通常再经过 Final Norm
→ Output Layer / LM Head
→ Logits
```

因此最后一个 Block 仍不直接选择下一个 Token。

## 框架层检查

1. 为什么 Block 2 能利用 Block 1 已经形成的信息？
2. 形状不变为什么不表示内容没变？
3. `intermediate_size` 会不会逐层累加成越来越宽的主干？
4. 最后一个 Block 输出以后还要经过什么？

能回答这四题，就可以进入 [[00-Output-Layer输出层概览|Output Layer]]。想继续理解内部流动时，再读 [[00-Block堆叠基础机制概览|Block 堆叠基础机制]]。
