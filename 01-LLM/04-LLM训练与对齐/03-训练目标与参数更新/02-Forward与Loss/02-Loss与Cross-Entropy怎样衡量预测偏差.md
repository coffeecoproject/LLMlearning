---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Forward与Loss概览|Forward 与 Loss 概览]]"
previous: "[[01-Forward怎样产生Hidden-States与Logits|Forward 怎样产生 Hidden States 与 Logits]]"
next: "[[03-从Token-Loss到整体训练目标|从 Token Loss 到整体训练目标]]"
tags: [llm, loss, cross-entropy, probability]
---

# Loss 与 Cross-Entropy 怎样衡量预测偏差

> [!summary]
> Cross-Entropy Loss 重点检查模型给正确目标 Token 分配了多少概率：正确目标概率越低，惩罚通常越大；多个有效位置的结果再按训练配方汇总。

## 为什么需要 Loss

模型输出的是大量候选分数：

```text
苹果：高
香蕉：中
汽车：低
……
```

训练系统需要一个可以持续比较的目标：

```text
当前预测有多偏离正确 Labels？
```

Loss 把这种偏差变成数值，供 Backward 计算参数变化方向。

## 一个三选一例子

> [!example] 教学示意
> 正确目标是“苹果”。

模型 A：

```text
苹果：0.80
香蕉：0.15
汽车：0.05
```

模型 B：

```text
苹果：0.10
香蕉：0.70
汽车：0.20
```

因为模型 B 给正确目标“苹果”的概率更低，所以它在这个位置的 Cross-Entropy Loss 更高。

## 为什么不直接使用“答对或答错”

假设两个模型最终最高概率都是“苹果”：

```text
模型C：苹果 0.51，香蕉 0.49
模型D：苹果 0.95，香蕉 0.05
```

如果只记 0 或 1：

```text
两者都答对
```

就看不到模型 D 对正确目标更有把握。Cross-Entropy 保留了概率分布中的差别，提供更连续的训练信号。

## Cross-Entropy 的直观含义

当前主线可以记成：

```text
正确 Token 概率高
→ 惩罚较小

正确 Token 概率低
→ 惩罚较大
```

模型对错误 Token 给出的概率也会间接影响正确 Token 的概率，因为所有候选经过 Softmax 后共同形成一个分布。

## 多个位置怎样汇总

假设一个 Batch 有三个有效目标位置：

```text
位置1 Loss：0.2
位置2 Loss：0.8
位置3 Loss：0.5
```

教学上可以用简单平均理解：

```text
(0.2 + 0.8 + 0.5) ÷ 3 = 0.5
```

真实训练可能使用平均、求和、样本权重、Token 权重或其他配方。Padding 与 `ignore_index` 位置不应进入普通平均分母。

PyTorch 的 `CrossEntropyLoss` 默认支持对非忽略目标求平均，并允许用 `ignore_index` 排除目标位置。

来源：[PyTorch CrossEntropyLoss 官方文档](https://docs.pytorch.org/docs/stable/generated/torch.nn.CrossEntropyLoss.html)，核对日期：2026-07-28。

## Loss 是不是越低越好

在同一数据、目标和计算方式下，Loss 下降通常说明模型更能预测这批数据。

但需要保留边界：

- 训练 Loss 低可能来自记忆或数据重复；
- 验证 Loss 可能开始变差，说明泛化不一定继续提升；
- 不同 Tokenizer、Vocabulary 或数据集的 Loss 不宜直接比较；
- 低 Next Token Loss 不自动等于事实可靠、无偏见或 Agent 执行安全；
- 后训练中的奖励目标可能与预训练 Cross-Entropy 不同。

## Loss 会告诉每个参数改多少吗

不会。

```text
Loss
→ 提供整体优化目标

Backward
→ 计算各参数对 Loss 的 Gradient

Optimizer
→ 决定实际更新量
```

Loss 是起点，不是完整更新指令。

## 可选数学直觉

单个正确目标常可简化理解为：

```text
Loss = -log(正确 Token 的概率)
```

不要求计算对数，只需观察：

```text
概率接近 1 → Loss 接近 0
概率接近 0 → Loss 快速变大
```

它会对“非常自信但预测错误”的情况给出较强惩罚。

## 常见误解

- **“Loss 就是错误率。”** Loss 会考虑概率信心，不只是对错计数。
- **“Loss 为零表示模型绝对正确。”** 它只对应指定训练目标和数据。
- **“不同模型的 Loss 数值可以直接比较。”** Tokenizer、数据和汇总方法不同会破坏可比性。
- **“Loss 会直接更新参数。”** 参数更新还需要 Backward 和 Optimizer。

## 理解检查

1. 两个模型都把正确 Token 排第一，为什么 Loss 仍可能不同？
2. Padding 位置为什么不应进入 Loss 平均？
3. 训练 Loss 下降为什么不能单独证明模型整体更可靠？

## 继续学习

- 上一篇：[[01-Forward怎样产生Hidden-States与Logits|Forward 怎样产生 Hidden States 与 Logits]]
- 下一篇：[[03-从Token-Loss到整体训练目标|从 Token Loss 到整体训练目标]]
