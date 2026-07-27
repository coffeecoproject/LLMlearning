---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[Normalization基础机制概览]]"
previous: "[[Pre-Norm与Post-Norm]]"
next: "[[Normalization小数字示例]]"
tags: [llm, normalization, final-norm, transformer-block]
---

# Block 内 Norm 与 Final Norm

> [!summary]
> Block 内 Norm 服务每一层 Attention 或 FFN 子层；Final Norm 位于整个 Block 堆叠之后，整理最终 Hidden States，再交给 LM Head。

## Block 内 Norm

在常见 Pre-Norm Decoder Block 中：

```text
Block 输入
→ 第一次 Norm
→ Attention
→ Residual
→ 第二次 Norm
→ FFN
→ Residual
→ Block 输出
```

如果模型有 36 个 Blocks，常见情况下每个 Block 都有对应的 Norm 组件，而不是整个模型只共享一对 Norm。

## Final Norm

所有 Blocks 结束后，许多现代 Decoder-only LLM 还有：

```text
最后一个 Block 输出
→ Final Norm
→ LM Head
→ Logits
```

它处理的是最后一层所有有效 Token 位置的 Hidden States，不是只处理最后一个 Token。

## 为什么 Pre-Norm 架构常有 Final Norm

Pre-Norm 在每个子层之前整理分支输入，但最后一次 Residual 相加后，主干结果不会自动再经过下一个 Block 的 Pre-Norm——因为已经没有下一个 Block。

因此堆叠结束后常加 Final Norm，为 LM Head 提供经过尺度整理的最终表示：

```text
最后一次 Residual 输出
→ Final Norm
→ LM Head
```

这是常见结构原因，不应扩展成“所有 Transformer 必须拥有完全相同的 Final Norm”。

## 参数是不是同一套

通常：

```text
Block 1 Norm 参数
≠ Block 2 Norm 参数
≠ Final Norm 参数
```

它们使用相同算法类型，不表示共享同一组 Weight。是否共享必须查看具体实现。

## Final Norm 属于哪里

从不同学习角度可以这样说：

```text
模型结构角度
→ 位于 Transformer Block 堆叠之后

输出数据流角度
→ 位于 LM Head 之前
```

所以本学习库在 Normalization 专题解释其机制，在 Output Layer 专题解释它怎样连接 Logits。

## 常见误解

- Final Norm 不等于“最后一个 Block 内的第二个 Norm”。
- Final Norm 不只处理最后一个 Token 位置。
- 多个 Norm 使用同一算法，不表示参数共享。
- Final Norm 还没有产生词表分数。

## 理解检查

1. Block 内 Norm 与 Final Norm 分别位于哪里？
2. 为什么 Pre-Norm 堆叠结束后常需要 Final Norm？
3. Final Norm 输出以后还需要哪一层产生 Logits？

下一篇：[[Normalization小数字示例|Normalization 小数字示例]]。
