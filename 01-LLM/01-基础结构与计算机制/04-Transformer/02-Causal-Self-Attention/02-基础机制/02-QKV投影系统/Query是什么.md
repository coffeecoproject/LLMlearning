---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[QKV投影系统概览]]"
previous: "[[为什么需要QKV三种角色]]"
next: "[[Key是什么]]"
tags: [llm, attention, query]
---

# Query 是什么

> [!summary]
> Query 是某个接收位置根据其当前 Hidden State 产生的匹配向量，用来与各个可见位置的 Key 计算相关分数。

## Query 属于哪个位置

序列中的每个位置通常都会产生自己的 Query：

```text
位置 0 的 Hidden State → Q₀
位置 1 的 Hidden State → Q₁
位置 2 的 Hidden State → Q₂
```

更新位置 `i` 时，主要使用该位置的 `Qᵢ` 去比较允许看见的各个 Key。

```text
Qᵢ 与 K₀ 比较
Qᵢ 与 K₁ 比较
……
Qᵢ 与 Kᵢ 比较
```

在 Causal Self-Attention 中，未来位置的 Key 会被 Mask 排除。

## “查询”不等于自然语言问题

把 Query 理解为“当前位置在找什么”有助于入门，但它不是一句可读的问题，例如：

```text
请帮我找到这句话中的主语。
```

真实 Query 是一组浮点数。它表达的是模型经过训练后形成的匹配方向，通常不能直接翻译成一句自然语言。

## Query 会随上下文变化吗

会。Query 来自当前层的 Hidden State，而 Hidden State 已经可能包含上下文信息：

```text
当前层 Hidden State 不同
→ 经过同一个 Q 投影
→ 得到的 Query 也不同
```

因此，同一个 Token 在不同句子、不同位置或不同模型层中，Query 可以不同。

这与 Token Embedding 的对象类型不同：Token Embedding 是模型保存的参数行；Query 是由当前 Hidden State 计算出来的临时表示。

## Query 自己不包含最终关注结果

仅看到 Query，还不知道它最终会关注哪里。必须把它与候选位置的 Key 比较，并继续经过 Scaling、Mask 和 Softmax：

```text
Query
+ 多个 Key
→ 多个 Attention Score
→ Attention Weight
```

所以 Query 是匹配的一侧，不是最终权重，也不是汇总后的上下文结果。

## 一个教学示意

假设单个 Head 的 Query 宽度为 3：

```text
Q₂ = [0.4, -0.1, 0.8]
```

这表示位置 2 当前用于匹配的三维向量。数字本身不表示：

```text
第 1 维 = 主语
第 2 维 = 时间
第 3 维 = 情绪
```

它们必须与 Key 一起参与计算才具有当前机制中的作用。

## 常见误解

- **“整句话只有一个 Query。”** Self-Attention 通常为每个位置、每个 Head 产生 Query。
- **“Query 是用户输入的问题。”** 它是模型内部向量，即使输入不是问句也会存在。
- **“Query 已经决定了最终注意位置。”** 还需要候选 Key、Mask 和 Softmax 等步骤。
- **“同一个 Token 的 Query 永远相同。”** Query 由当前层 Hidden State 产生，会随上下文和层变化。

## 理解检查

1. 更新位置 `i` 时，使用哪个位置的 Query？
2. 为什么同一 Token 在不同上下文中的 Query 可以不同？
3. 为什么只知道 Query 还不能知道最终 Attention Weight？

下一篇：[[Key是什么|Key 是什么]]。
