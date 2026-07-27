---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN概览|FFN概览]]"
previous: "[[00-FFN概览|FFN概览]]"
next: "[[00-FFN基础机制概览|FFN基础机制概览]]"
tags: [llm, ffn, beginner, framework]
---

# FFN 一页看懂

> [!summary]
> FFN 是 Transformer Block 内部的“逐位置加工环节”：它不负责让 Token 互相读取，而是继续处理 Attention 已经汇集好的上下文信息。

## 先看它在哪里

```text
多个 Token 位置的 Hidden State
            ↓
        Attention
   不同位置之间交流信息
            ↓
          FFN
    每个位置分别继续加工
            ↓
    下一个 Transformer Block
```

准确地说，Attention 和 FFN 周围还有 Norm 与 Residual。这里只展示两者的核心分工。

## FFN 只需先回答五个问题

### 1. 它在哪里？

在 Transformer Block 内部。常见结构中，每个 Block 都有 Attention 和 FFN。

### 2. 它接收什么？

接收每个 Token 位置当前的 **Hidden State**，也就是模型内部正在不断更新的向量表示。

它不直接接收原始文字，也不直接接收 Token ID。

### 3. 它做什么？

同一层 FFN 使用同一套已经训练好的规则，分别加工每个位置的向量：

```text
位置 1 的向量 → 同一套 FFN → 位置 1 的变化
位置 2 的向量 → 同一套 FFN → 位置 2 的变化
位置 3 的向量 → 同一套 FFN → 位置 3 的变化
```

各位置输入不同，所以即使使用同一套规则，输出也会不同。

### 4. 为什么已经有 Attention，还需要 FFN？

因为两者解决的问题不同：

```text
Attention：从其他允许读取的位置取得哪些信息
FFN：怎样继续加工当前位置已经取得的信息
```

只有 Attention，模型能在位置之间传递信息，但缺少 Block 内另一类重要的特征变换。

### 5. 它输出什么？

FFN 输出的是一个与主干同样宽的“变化向量”。这个变化通常通过 Residual 加回原有 Hidden State，然后继续进入下一个 Block。

它不会直接输出下一个 Token；产生词表分数还需要后面的 Final Norm 与 Output Layer。

## 一个直白例子

假设一句话中有“苹果”和“股票”。Attention 可以把“股票”的上下文影响带到“苹果”所在位置，使该位置不再只表示水果含义。

随后 FFN 加工的是这个已经带有上下文的表示，而不是孤立的“苹果”两个字：

```text
“苹果”位置的原有表示
+ Attention 带来的“股票”等上下文影响
→ 当前 Hidden State
→ FFN 继续加工
```

> [!warning] 类比边界
> 真实模型处理的是 Token 位置上的数值向量；实际 Tokenizer 也不一定把“苹果”切成一个 Token。例子只用于说明 Attention 与 FFN 的先后关系。

## 暂时不需要学什么

如果你现在只想理解大模型框架，可以先跳过：

- `hidden_size` 和 `intermediate_size` 的具体形状；
- 投影矩阵和参数量计算；
- SwiGLU、GeGLU；
- MoE、Router、Expert；
- 不同开放模型的配置差异。

这些都不会改变 FFN 的核心位置与作用。

## 检查自己是否已经理解

尝试用自己的话回答：

1. FFN 在 Transformer Block 内还是 Block 外？
2. Attention 与 FFN 最核心的分工是什么？
3. FFN 的输入是原始文字、Token ID，还是 Hidden State？
4. FFN 会直接选出下一个 Token 吗？

能回答这四题，就已经掌握 FFN 的框架层知识，可以先继续学习 Transformer 后续结构。

想理解内部机制时再进入：[[00-FFN基础机制概览|FFN 基础机制]]。
