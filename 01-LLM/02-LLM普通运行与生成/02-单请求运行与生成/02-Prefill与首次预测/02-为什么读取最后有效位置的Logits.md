---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Prefill与首次预测概览|Prefill与首次预测概览]]"
previous: "[[01-Prefill是什么|Prefill是什么]]"
next: "[[00-KV-Cache与增量Decode概览|KV-Cache与增量Decode概览]]"
tags: [llm, inference, logits, causal-lm, sequence-length]
---

# 为什么读取最后有效位置的 Logits

> [!summary]
> Decoder-only 模型会为输入序列的每个位置产生 Logits；要继续整段序列，应读取最后一个真实 Token 位置的 Logits，因为它包含当前完整可见上下文。

## Output Layer 实际输出了什么

假设输入有 4 个 Token，Output Layer 的逻辑形状是：

```text
[sequence_length, vocab_size]
= [4, vocab_size]
```

也就是四个位置各有一组词表分数：

```text
位置 1 的 Logits：根据 Token 1 预测后续
位置 2 的 Logits：根据 Token 1~2 预测后续
位置 3 的 Logits：根据 Token 1~3 预测后续
位置 4 的 Logits：根据 Token 1~4 预测下一个 Token
```

普通续写需要最后一组，因为它使用了当前完整上下文。

## 为什么说“最后有效位置”而不是“数组最后位置”

批处理时，短序列可能被 Padding 对齐：

```text
序列 A：[我] [喜欢] [猫] [PAD] [PAD]
序列 B：[今] [天] [天] [气] [好]
```

对序列 A，真正最后位置是 `[猫]`，不是数组末尾的 `[PAD]`。Runtime 需要根据有效长度或 Mask 找到对应位置。

## 这和训练有什么区别

> [!info] 阶段区分
> 训练 Causal LM 时，通常会利用多个有效位置的 Logits 与各自的“下一个 Token”目标计算 Loss；普通生成时，当前只需要最后有效位置的 Logits 来决定下一步。

这里提到训练只是为了划清差异，不展开 Loss 的计算。

## 常见误解

- 模型不是只为最后位置做了 Output Layer；常见实现会得到多个位置的 Logits，只是生成控制器选取当前需要的一组。
- “最后位置”不是固定数组下标，Padding 会改变物理排列。
- 最后位置 Logits 表示下一个 Token 的候选强弱，不是整个回答的所有 Token。

## 理解检查

1. 为什么最后有效位置包含的可见上下文最多？
2. 有 Padding 时，为什么不能盲目取数组最后一项？
3. 普通生成为什么不需要同时使用所有位置的 Logits？

下一部分：[[00-KV-Cache与增量Decode概览|KV Cache 与增量 Decode]]。
