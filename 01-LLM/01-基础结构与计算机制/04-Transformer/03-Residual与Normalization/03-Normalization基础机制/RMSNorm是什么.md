---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[Normalization基础机制概览]]"
previous: "[[LayerNorm是什么]]"
next: "[[LayerNorm与RMSNorm对比]]"
tags: [llm, rmsnorm, normalization, qwen3]
---

# RMSNorm 是什么

> [!summary]
> RMSNorm 使用向量各维度平方的平均值衡量整体幅度，再按其均方根缩放；它通常不执行 LayerNorm 的“先减去均值”步骤。

## RMS 是什么意思

`RMS` 是 Root Mean Square，中文常译为均方根：

```text
各维度先平方
→ 求平均
→ 开平方
```

它给出向量整体数值幅度的一种衡量。

## 主要过程

对某个 Token 位置的 Hidden State：

```text
x
→ 计算 RMS 尺度
→ x 除以该尺度
→ 乘以可学习 weight / gamma
→ 输出
```

与常见 LayerNorm 相比，RMSNorm 通常：

- 不先减去均值；
- 不使用同类 Bias 参数；
- 保留一个按维度学习的 Scale。

具体实现仍应查看对应模型代码。

## 可选技术轮廓

```text
RMS(x)
= sqrt(mean(x²) + eps)

输出
= x / RMS(x) × gamma
```

公式只用于指出它根据平方幅度缩放，而不是根据均值与方差完整居中。

## 为什么现代 LLM 常使用它

RMSNorm 结构更简洁，省略了均值中心化步骤，同时在许多 Transformer 模型中能提供有效的尺度控制。

这不表示 RMSNorm 在所有模型、所有训练设置下都必然优于 LayerNorm。选择取决于架构和经过验证的训练方案。

## Qwen3-8B 观察

Qwen3-8B 官方配置包含：

```json
{
  "rms_norm_eps": 1e-06
}
```

官方实现中的 `Qwen3RMSNorm` 使用输入平方均值、`rsqrt`、`eps` 和可学习 Weight 完成缩放。模型层包含 Attention 前和 FFN 前的 RMSNorm，堆叠结束后还有 Final RMSNorm。

> [!source]
> 来源：[Qwen3-8B 官方配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)、[Qwen3 官方实现](https://github.com/huggingface/transformers/blob/main/src/transformers/models/qwen3/modeling_qwen3.py)。核对日期：2026-07-27。

## 它仍然是逐位置处理

RMSNorm 与 LayerNorm 一样，通常对各 Token 位置的 Hidden State 分别处理：

```text
位置 A → 计算 A 自己的 RMS
位置 B → 计算 B 自己的 RMS
```

因此它不负责把位置 A 的内容传给位置 B。

## 常见误解

- RMSNorm 不等于 LayerNorm 改了一个名字。
- 它通常不减均值，但仍会控制整体尺度。
- `rms_norm_eps` 不是上下文长度或训练步数。
- 使用 RMSNorm 不代表模型没有可学习 Norm 参数。

## 理解检查

1. RMSNorm 怎样衡量向量整体幅度？
2. 它与 LayerNorm 最直观的计算差别是什么？
3. Qwen3 的 `rms_norm_eps` 用于什么？

下一篇：[[LayerNorm与RMSNorm对比|LayerNorm 与 RMSNorm 对比]]。
