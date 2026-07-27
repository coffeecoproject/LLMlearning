---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[Block堆叠基础机制概览]]"
previous: "[[为什么主干形状通常保持不变]]"
next: "[[Block参数与边界概览]]"
tags: [llm, final-hidden-state, final-norm, output-layer]
---

# Final Hidden State 怎样交给输出层

> [!summary]
> 最后一个 Transformer Block 输出整条序列的 Final Hidden States；许多 Decoder-only LLM 会再执行 Final Norm，然后由 LM Head 把每个位置映射成词表 Logits。

## 从最后一个 Block 开始

假设模型有 `N` 个 Blocks：

```text
Hᴺ⁻¹
→ Block N
→ Hᴺ
```

`Hᴺ` 就是 Transformer 主体最后得到的一组 Hidden States。

## Final 不表示只有一个位置

如果输入序列有 4 个 Token，Final Hidden States 通常仍包含 4 个位置：

```text
[sequence_length, hidden_size]
= [4, hidden_size]
```

`Final` 表示“最后一层”，不是“只剩最后一个 Token”。

## Final Norm 位于哪里

许多现代 Decoder-only LLM 的主线是：

```text
最后一个 Block 输出
→ Final Norm
→ LM Head
→ Logits
```

Final Norm 不属于每个 Block 内的那一次 Norm，而是整个 Block 堆叠结束后的额外归一化步骤。具体模型是否存在、采用 LayerNorm 还是 RMSNorm，应查看对应架构。

## LM Head 做什么

LM Head 把每个位置从 `hidden_size` 映射到 `vocab_size`：

```text
Final Hidden State
→ LM Head
→ 每个词表 Token 的 Logit
```

所以：

```text
Final Hidden State
≠ 概率
≠ 下一个 Token
```

它仍是模型内部表示。

## 为什么运行时常看最后一个位置

普通自回归生成需要预测“当前序列之后”的下一个 Token，因此运行时通常使用最后一个有效位置对应的 Logits 进行下一步选择。

但模型前向结构可以为序列中的各位置产生 Logits；训练阶段通常会让多个有效位置参与 next-token Loss。这个训练与运行差异将在 Output Layer 和后续阶段模块中展开。

## 与最后一个 Token 的 Hidden State 区分

```text
Final Hidden States
→ 最后一层所有位置的表示

last-position final hidden state
→ 最后一层中最后一个有效位置的表示
```

一个强调层，一个强调序列位置，不能因为都出现“最后”就混在一起。

## 常见误解

- 最后一层不表示只保留一个 Token 位置。
- Final Hidden States 还没有直接变成文字。
- Final Norm 不是 Tokenizer 的 Normalization。
- 运行时使用最后位置预测，不等于训练时只有最后位置参与学习。

## 理解检查

1. Final Hidden States 中的 `Final` 指层还是 Token 位置？
2. 为什么还需要 LM Head 才能得到词表 Logits？
3. “最后一层所有位置”与“最后一个位置”有什么区别？

基础机制完成。需要继续理解层数和参数时，进入 [[Block参数与边界概览|Block 参数与边界]]；否则可以进入 [[Output-Layer输出层概览|Output Layer]]。
