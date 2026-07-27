---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[Value加权与Context-Mixing概览]]"
previous: "[[Value加权求和怎样计算]]"
next: "[[为什么每个位置得到不同的Context]]"
tags: [llm, attention, context-vector, hidden-state]
---

# Context Vector 是什么

> [!summary]
> Context Vector 是一个接收位置在当前 Attention Head 中，对所有允许来源的 Value 完成加权求和后得到的上下文汇总向量。

## 它从哪里来

```text
各来源位置的 Value
+ 当前接收位置对应的一组 Attention Weights
→ Value 加权求和
→ Context Vector
```

例如：

```text
0.2V₀ + 0.7V₁ + 0.1V₂
→ 当前接收位置的 Context Vector
```

这里的 Context Vector 不是额外查表得到的，也不是一个单独输入。它是本次 Attention 计算产生的结果。

## “Context”具体指什么

这里的上下文不是原始文字列表，而是当前位置从可见来源 Value 中汇总出的内部表示。

它回答的是：

> 对当前接收位置而言，在这个层、这个 Head 的当前计算中，可见来源提供的信息混合后形成了怎样的向量？

因此，同一段文本中不同接收位置会有自己的 Context Vector。

## 它与 Value 的区别

```text
Value
→ 一个来源位置准备提供的内容表示

Context Vector
→ 一个接收位置从多个来源 Value 汇总后的结果
```

Value 站在“来源”的角度；Context Vector 站在“接收”的角度。

## 它与 Hidden State 的区别

Context Vector 与 Hidden State 都是向量，但处在不同位置：

```text
输入 Hidden States
→ 投影成 Q、K、V
→ 单个 Head 产生 Context Vectors
→ 多个 Head 合并并做 Output Projection
→ 与 Residual 等路径继续组合
→ 形成后续 Hidden States
```

所以本节中的单头 Context Vector 只是 Attention 子层内部的中间结果，不能直接等同于完整的新 Hidden State。

## 它与 Attention Weight 的区别

| 对象 | 表示什么 | 常见形式 |
|---|---|---|
| Attention Weight | 从各来源取多少 | 一组标量系数 |
| Value | 每个来源提供什么 | 每个来源一个向量 |
| Context Vector | 汇总后得到什么 | 当前接收位置一个向量 |

即使 Weight 形如 `[0.2, 0.7, 0.1]`，Context Vector 也可能是数百或数千维的内部向量；二者不能互换。

## Context Vector 是“完整语义”吗

不能这样理解。它只反映：

- 某一层；
- 某一个 Head；
- 某一个接收位置；
- 本次 Value 汇总的结果。

模型后面还有其他 Head、Output Projection、Residual、Normalization、FFN 和更多 Transformer Blocks。单独观察一个 Context Vector，不能还原模型的完整意图或最终答案原因。

## 命名提醒

不同资料可能把这一步称为：

- `Context Vector`；
- `Attention Output`；
- `Head Output`。

这些称呼有时指代范围略有不同。本套笔记用 `Context Vector` 指**单个 Head 完成 Value 加权求和后的结果**，用后续章节说明多头合并后的 Attention 子层输出。

## 常见误解

- **“Context Vector 就是上下文原文的压缩包。”** 它是计算得到的连续向量，不能直接无损还原原文。
- **“它就是新的 Hidden State。”** 它还要经过多头合并、输出投影和残差等步骤。
- **“一个序列只有一个 Context Vector。”** 每个接收位置、每个 Head 都有自己的结果。
- **“它已经决定了下一个 Token。”** 输出词表分数属于后面的 Output Layer。

## 理解检查

1. Value 与 Context Vector 分别站在来源还是接收角度？
2. 为什么单头 Context Vector 不能直接等同于新 Hidden State？
3. 一段包含多个 Token 的序列是否只有一个 Context Vector？

下一篇：[[为什么每个位置得到不同的Context|为什么每个位置得到不同的 Context]]。
