---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[Normalization基础机制概览]]"
previous: "[[RMSNorm是什么]]"
next: "[[Pre-Norm与Post-Norm]]"
tags: [llm, layernorm, rmsnorm, comparison]
---

# LayerNorm 与 RMSNorm 对比

> [!summary]
> 两者都按 Token 位置整理 Hidden State 尺度；LayerNorm 通常先减均值并按方差缩放，RMSNorm 通常直接按均方根幅度缩放。

## 一张表看清

| 问题 | LayerNorm | RMSNorm |
|---|---|---|
| 是否通常减去均值 | 是 | 否 |
| 主要尺度统计 | 方差 / 标准差 | 均方根 RMS |
| 可学习 Scale | 通常有 | 通常有 |
| 可学习 Bias | 常见但实现可省略 | 通常没有同类 Bias |
| 是否跨 Token 混合 | 否 | 否 |
| 是否改变主干宽度 | 否 | 否 |

## 相同点

二者通常都：

- 对每个 Token 位置分别处理；
- 沿 `hidden_size` 维度计算；
- 保持输入输出形状一致；
- 在训练和运行前向中执行；
- 使用 `eps` 避免分母过小；
- 可以位于 Attention、FFN 或 Final Output 之前。

## 核心差别

```text
LayerNorm
→ 先把数值围绕均值重新居中
→ 再控制分散尺度

RMSNorm
→ 不做同样的中心化
→ 直接控制整体平方幅度
```

因此两者对同一个输入通常产生不同数值，但都可以作为 Transformer 的尺度管理机制。

## 不要从名称推断位置

`LayerNorm` 与 `RMSNorm` 描述“怎样算”；`Pre-Norm` 与 `Post-Norm` 描述“Norm 放在哪里”。

例如：

```text
Pre-Norm + RMSNorm
Pre-Norm + LayerNorm
Post-Norm + LayerNorm
```

这些组合在概念上都可能存在。算法类型与结构位置是两个维度。

## 怎样判断真实模型

1. 查官方配置是否存在 `rms_norm_eps`、`layer_norm_eps` 等字段；
2. 查看模型实现使用哪个 Norm 类；
3. 查看 Block 中调用发生在子层之前还是之后；
4. 查看堆叠结束后是否还有 Final Norm。

不能只凭模型名称或宣传页猜测。

## 常见误解

- RMSNorm 不是一种 Attention 变体。
- LayerNorm 的 Layer 不表示只在层与层之间出现一次。
- 使用 RMSNorm 不自动证明模型更强。
- Pre-Norm 不等于 RMSNorm，Post-Norm 也不等于 LayerNorm。

## 理解检查

1. LayerNorm 与 RMSNorm 在“是否减均值”上有什么不同？
2. 为什么 Norm 算法类型与 Norm 放置位置必须分开？
3. 两者为什么都不会完成 Token 间信息混合？

下一篇：[[Pre-Norm与Post-Norm|Pre-Norm 与 Post-Norm]]。
