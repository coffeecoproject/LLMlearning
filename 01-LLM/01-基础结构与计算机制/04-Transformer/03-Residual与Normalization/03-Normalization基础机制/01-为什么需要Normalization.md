---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Normalization基础机制概览|Normalization基础机制概览]]"
previous: "[[00-Normalization基础机制概览|Normalization基础机制概览]]"
next: "[[02-LayerNorm是什么|LayerNorm是什么]]"
tags: [llm, normalization, numerical-scale, transformer]
---

# 为什么需要 Normalization

> [!summary]
> Transformer 会连续堆叠许多 Attention、FFN 和 Residual 更新；Normalization 让进入关键子层的向量尺度保持在更可控的范围，帮助深层计算与训练保持稳定。

## 问题从哪里来

每个 Block 都会更新 Hidden State：

```text
旧表示
+ Attention 变化
+ FFN 变化
→ 新表示
```

继续堆叠后，各维度数值的整体尺度可能不断变化。过大或过小的数值会影响后续投影、激活函数、Attention Score 和训练梯度。

Normalization 提供一个相对稳定的数值接口：

```text
当前 Hidden State
→ 整理尺度
→ 再进入 Attention 或 FFN
```

## 它整理的是什么

假设一个 Token 位置的 Hidden State 是：

```text
[2.0, -1.0, 5.0, 0.5]
```

LayerNorm 或 RMSNorm 会使用这一位置向量内部各维度的统计量做缩放。对常见 Transformer Hidden State：

```text
每个 Token 位置分别计算
→ 沿 hidden_size 维度整理
```

它通常不会把整句话所有 Token 混在一起求一个共同平均，也不依赖其他用户请求组成的 Batch 才能工作。

## 为什么不是把所有向量变得一样

Normalization 使用相同规则处理不同输入，但统计量来自每个位置自身：

```text
位置 A 的向量 → 使用 A 自己的统计量
位置 B 的向量 → 使用 B 自己的统计量
```

不同向量的方向和相对数值关系仍然可以不同，后续还有可学习 Scale 等参数。因此它不是把所有 Token 压成同一个标准答案。

## 它与 Residual 的分工

```text
Residual
→ 保留主干并加入子层变化

Normalization
→ 整理送入或离开子层时的数值尺度
```

二者经常一起出现，是因为深层模型既要保持信息通路，也要控制数值条件。

## 训练与运行

> [!info] 两阶段共同
> 训练和运行都会执行相同架构规定的 Normalization 前向计算。

> [!info] LLM 训练阶段
> Norm 的可学习 Scale，以及 LayerNorm 中可能存在的 Bias，会根据梯度更新。Normalization 也影响深层网络的优化稳定性。

> [!info] LLM 运行阶段
> 使用训练好的 Norm 参数处理当前 Hidden States，不会因为一次聊天重新训练这些参数。

## 不要过度承诺

Normalization 有助于稳定计算与优化，但不能简单理解成：

```text
有了 Norm
→ 数值永远不会异常
→ 模型训练一定成功
```

训练稳定性还与初始化、学习率、精度、Residual 结构、数据和优化器等因素有关。

## 常见误解

- Normalization 处理的是向量数值，不是原始文本格式。
- 它通常按 Token 位置分别处理，不是跨 Batch 用户求平均。
- 它不会让所有 Token 的 Hidden State 完全相同。
- 它在运行时仍然执行，不是只为反向传播存在。

## 理解检查

1. 为什么多层 Residual 更新会让数值尺度成为问题？
2. Transformer Norm 通常沿哪个维度计算？
3. 为什么使用相同 Norm 规则不会让所有 Token 表示相同？

下一篇：[[02-LayerNorm是什么|LayerNorm 是什么]]。
