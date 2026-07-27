---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Residual与Normalization概览]]"
previous: "[[Residual怎样连接Attention与FFN]]"
next: "[[为什么需要Normalization]]"
tags: [llm, normalization, layernorm, rmsnorm]
---

# Normalization 基础机制

> [!summary]
> Normalization 在每个 Token 位置内部整理 Hidden State 的数值尺度，让 Attention、FFN 和深层 Residual 主干接收到更稳定的数值表示。

## 阅读顺序

1. [[为什么需要Normalization|为什么需要 Normalization]]；
2. [[LayerNorm是什么|LayerNorm 是什么]]；
3. [[RMSNorm是什么|RMSNorm 是什么]]；
4. [[LayerNorm与RMSNorm对比|LayerNorm 与 RMSNorm 对比]]；
5. [[Pre-Norm与Post-Norm|Pre-Norm 与 Post-Norm]]；
6. [[Block内Norm与Final-Norm|Block 内 Norm 与 Final Norm]]；
7. [[Normalization小数字示例|Normalization 小数字示例]]。

## 基础路线

```text
一个 Token 位置的 Hidden State
→ 计算该向量自身的尺度统计
→ 按对应规则重新缩放
→ 可学习 Scale 等参数再次调节
→ 交给 Attention、FFN 或 Output Layer
```

## 先划清范围

Transformer 中的 Normalization 通常不是：

- 修改原始文字格式；
- 跨所有用户请求求平均；
- 把所有 Token 变成相同向量；
- 保存长期记忆；
- 直接选择下一个 Token。

它主要处理每个位置 Hidden State 内部各维度的数值关系。

下一篇：[[为什么需要Normalization|为什么需要 Normalization]]。
