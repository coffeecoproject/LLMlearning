---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer基础机制概览|Output-Layer基础机制概览]]"
previous: "[[02-LM-Head是什么|LM-Head是什么]]"
next: "[[04-Softmax怎样把分数变成概率|Softmax怎样把分数变成概率]]"
tags: [llm, logits, scores, vocabulary]
---

# Logits 是什么

> [!summary]
> Logits 是模型为候选 Token 给出的原始数值分数；它们可以比较相对高低，但还不是总和为 1 的概率。

## 观察一个教学例子

假设词表只有四项：

```text
候选 Token： 猫     狗     睡觉    <EOS>
Logits：     1.8    0.2    2.5     -0.6
```

可以先读出：在当前上下文和当前模型参数下，`睡觉` 的原始分数最高，`<EOS>` 最低。

但不能读成：

```text
睡觉的概率 = 2.5
```

概率通常位于 0 到 1 之间且总和为 1，而 Logits 可以为负数、零或任意正数，也不要求总和固定。

## Logit 的绝对值重要吗

对候选选择更重要的是彼此之间的相对差距。例如给全部 Logits 加上同一个常数，Softmax 后的概率不会改变：

```text
[2, 1, 0]
与
[12, 11, 10]
```

二者的相对差距相同，因此会形成相同的 Softmax 概率。这说明 Logit 不是一个可单独解释的“正确率分数”。

## 每个位置都有一组 Logits

如果序列长度为 3、词表大小为 5，那么单条序列的完整输出可以看作：

```text
位置 1 → 5 个 Logits
位置 2 → 5 个 Logits
位置 3 → 5 个 Logits

整体形状：[3,5]
```

加入 Batch 后通常是：

```text
[batch_size, sequence_length, vocab_size]
```

## Logits 是什么边界

在模型架构层，Logits 是一个很稳定的输出边界：Transformer 主体和 LM Head 已经完成前向计算，训练目标或生成策略可以开始使用结果。

但用户通过聊天 API 得到的通常是已经生成并 Decode 后的文字，不一定能直接看到完整 Logits。

## 常见误解

- 最高 Logit 不表示候选内容在现实世界中一定正确。
- Logit 为负不等于概率为负。
- Logits 不是 Attention Score；两者都叫分数，但处在不同环节、面向不同对象。
- 一个 Token 的 Logit 不是固定值，它会随上下文、位置和模型参数变化。

## 理解检查

1. 为什么 `[2,1,0]` 不能直接叫概率？
2. Logit 为负是否意味着候选不可能被选择？
3. Logits 与 Attention Score 分别在给什么对象打分？

下一篇：[[04-Softmax怎样把分数变成概率|Softmax 怎样把分数变成概率]]。
