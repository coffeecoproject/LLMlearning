---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Causal-Self-Attention概览]]"
previous: "[[Causal-Self-Attention概览]]"
next: "[[Attention基础机制概览]]"
tags: [llm, attention, causal-attention, framework, beginner]
---

# Attention 一页看懂

> [!summary]
> Attention 让每个 Token 位置根据当前需要，从允许读取的其他位置汇集不同份量的信息，从而形成上下文相关的表示。

## 为什么需要 Attention

Embedding 后，每个位置主要拥有自己的初始表示。要理解：

```text
我买了苹果公司的股票
```

“苹果”位置需要利用“公司”“股票”等上下文。Attention 就是 Block 中负责位置间信息交流的环节。

## 它接收和输出什么

```text
输入：所有 Token 位置当前的 Hidden States
        ↓
Attention：判断当前位置应从哪些可见位置取多少信息
        ↓
输出：每个位置得到一份上下文变化
```

Attention 输出仍是 Hidden State 主干上的向量变化，不是文字、Token ID 或最终概率。

## Q、K、V 先怎样理解

无需计算公式，先把三个角色读成：

```text
Query：当前位置正在寻找什么
Key：每个可见位置提供什么匹配线索
Value：匹配后真正汇入的信息内容
```

模型用 Query 与 Key 决定信息份量，再按这些份量汇总 Value。

> [!warning] 类比边界
> Q、K、V 都是 Hidden State 经过不同参数投影得到的数值向量，不是模型内部出现了三句自然语言问题。

## Causal 是什么意思

Decoder-only LLM 中，当前位置不能读取未来 Token：

```text
当前位置
→ 可以看自己和过去
→ 不能看未来
```

这个约束在训练和运行时都要保持，否则训练时会偷看答案，运行时的依赖关系也会与训练不一致。

## 它与 FFN 的分工

```text
Attention：不同位置之间交流
FFN：每个位置分别加工交流后的表示
```

二者都位于 Transformer Block 内部，不能互相替代。

## 框架层检查

1. Attention 为什么需要同时读取多个位置？
2. Q、K、V 分别承担什么角色？
3. Causal 约束限制的是什么？
4. Attention 输出为什么不是最终 Token？

能回答这四题，就可以进入 [[Residual与Normalization框架速览概览|Residual 与 Normalization 一页看懂]]。想了解完整计算链时，再读 [[Attention基础机制概览|Attention 基础机制]]。
