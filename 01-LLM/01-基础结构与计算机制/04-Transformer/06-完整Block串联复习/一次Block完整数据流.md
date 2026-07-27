---
type: review
module: 1
status: complete
audience: non-specialist
parent: "[[完整Transformer-Block串联复习概览]]"
previous: "[[完整Transformer-Block串联复习概览]]"
next: "[[完整Block形状追踪]]"
tags: [llm, transformer-block, attention, ffn, residual, normalization]
---

# 一次 Block 完整数据流

> [!summary]
> 一个常见串行 Pre-Norm Transformer Block 会先对主干表示做 Norm 和 Attention，再通过 Residual 加回；随后再做 Norm 和 FFN，并通过第二次 Residual 得到 Block 输出。

## 第 0 步：Block 接收什么

第一个 Block 接收 Embedding 形成的初始 Hidden States；如果模型采用 Absolute Position Embedding，位置表示通常已在输入附近结合。RoPE 或 ALiBi 则要到 Attention 内部才分别作用于 Q/K 或 Score。后续 Block 接收上一 Block 的输出。

统一记作：

```text
X = 当前 Block 输入 Hidden States
```

它通常包含整条序列各 Token 位置的向量。

## 第 1 步：Attention 前 Normalization

```text
X
→ Norm 1
→ X_norm
```

Norm 按每个 Token 位置整理其 `hidden_size` 维向量的数值尺度，不在 Token 位置之间传递信息。

Pre-Norm 中，Residual 主干仍然保留原始 `X`。

## 第 2 步：Causal Self-Attention

```text
X_norm
→ 产生 Q、K、V
→ 注入相应位置影响
→ QK 形成 Score
→ Scaling / Mask / Softmax
→ Weight 加权汇总 Value
→ 多 Head 合并
→ Output Projection
→ Attention 变化量
```

Causal Mask 保证每个位置只能使用允许读取的自己和过去位置。

Attention 子层完成的是位置之间的信息交流，但输出仍是数值向量，不是 Token 或答案。

## 第 3 步：第一次 Residual

```text
A = X + Attention(Norm(X))
```

这里：

```text
X
→ 原有主干

Attention(Norm(X))
→ Attention 分支建议加入的变化

A
→ Attention 更新后的中间 Hidden States
```

## 第 4 步：FFN 前 Normalization

```text
A
→ Norm 2
→ A_norm
```

第二个 Norm 服务 FFN 分支。它通常拥有自己的可学习参数，不等于重复使用第一个 Norm 的同一组 Weight。

## 第 5 步：FFN

基础路线：

```text
A_norm
→ 从 hidden_size 展开到 intermediate_size
→ Activation 或 Gate
→ 压回 hidden_size
→ FFN 变化量
```

FFN 对每个 Token 位置分别应用同一层的参数。它不在该子层内部重新读取其他 Token，但输入已经经过 Attention，因而包含上下文影响。

若模型采用 MoE，这里可能变成 Router 选择少数 Expert FFN；Block 的整体输入输出接口仍需回到主干宽度。

## 第 6 步：第二次 Residual

```text
Y = A + FFN(Norm(A))
```

`Y` 是当前 Block 的最终输出 Hidden States：

```text
当前 Block 输出 Y
= 下一 Block 输入 X
```

## 把六步压缩成两行

```text
A = X + Attention(Norm₁(X))
Y = A + FFN(Norm₂(A))
```

这两行是常见串行 Pre-Norm Block 的结构摘要，不是所有 Transformer 的唯一公式。

## 多个 Block 以后

```text
H⁰ → Block 1 → H¹
H¹ → Block 2 → H²
……
Hᴺ⁻¹ → Block N → Hᴺ
```

在常见 Pre-Norm Decoder-only LLM 中：

```text
Hᴺ
→ Final Norm
→ LM Head
→ 词表 Logits
```

模型运行阶段还会根据 Logits 继续选择 Token；训练阶段会根据 Logits 计算 Loss。这些属于 Output Layer 和后续阶段。

## 阶段边界

> [!info] 两阶段共同
> 上述 Block 前向数据流在 LLM 训练和运行时都会执行。

> [!info] LLM 训练阶段
> 完整训练序列使用 Causal Mask 防止未来答案泄漏，前向结果随后参与 Loss 和反向传播。

> [!info] LLM 运行阶段
> 参数固定。Prefill 和逐 Token Decode 都会调用这些 Block；KV Cache 可以复用 Attention 的历史 K/V，但不会取消 FFN、Residual 和 Norm。

## 架构变体提醒

真实模型可能采用：

- Post-Norm；
- 并行 Attention 与 FFN；
- RMSNorm 或 LayerNorm；
- MHA、GQA、MQA 或 MLA；
- Dense FFN 或 MoE；
- 额外 Q/K Norm、Residual 缩放等结构。

变体会改变内部路线，但“输入表示经过多层参数化更新，最终形成 Hidden States”这条主线仍成立。

## 理解检查

1. 两次 Norm 分别服务哪个子层？
2. 两次 Residual 分别把什么变化加回主干？
3. Attention 与 FFN 为什么不能互相替代？
4. 一个 Block 输出怎样进入下一 Block？

下一篇：[[完整Block形状追踪|完整 Block 形状追踪]]。
