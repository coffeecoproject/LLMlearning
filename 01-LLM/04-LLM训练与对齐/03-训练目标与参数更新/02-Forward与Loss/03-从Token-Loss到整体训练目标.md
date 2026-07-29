---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Forward与Loss概览|Forward 与 Loss 概览]]"
previous: "[[02-Loss与Cross-Entropy怎样衡量预测偏差|Loss 与 Cross-Entropy 怎样衡量预测偏差]]"
next: "[[../03-Backward与参数更新/00-Backward与参数更新概览|Backward 与参数更新概览]]"
tags: [llm, training-objective, token-loss, batch-loss]
---

# 从 Token Loss 到整体训练目标

> [!summary] 一句话理解
> 一个预测位置产生一个 Token Loss；训练程序汇总当前 Batch 中许多有效位置的 Loss，并通过持续更换 Batch，让参数逐渐服务于整个训练数据的目标。

## 所属阶段

**训练阶段。** 普通运行会计算下一个 Token 的概率，但不会为了更新参数而汇总训练 Loss。

## 为什么还要把 Loss 分层

模型不是只学习一个答案，而是在海量文本的许多位置上反复学习“下一个 Token 应该是什么”：

```text
一个预测位置
→ 一条序列的许多位置
→ 一个 Batch 的许多序列
→ 训练过程中的许多 Batch
```

不区分这些层次，就容易把“某一个 Token 预测得更好”误认为“整个模型已经变好”。

## 第一步：每个有效位置产生 Token Loss

假设真实的下一个 Token 是“苹果”：

```text
模型给“苹果”较高概率 → Token Loss 较低
模型给“苹果”较低概率 → Token Loss 较高
```

一条序列中通常有多个预测位置：

```text
我       → 喜欢
我 喜欢  → 苹果
我 喜欢 苹果 → <EOS>
```

每个需要学习的位置都能产生一个 Token Loss。Padding 或明确被屏蔽的位置不会参与普通汇总。

## 第二步：汇总成当前 Batch Loss

假设一个教学 Batch 只有三个有效位置，其 Loss 分别是：

```text
0.2、0.8、0.5
```

用最简单的平均方式：

```text
Batch Loss = (0.2 + 0.8 + 0.5) ÷ 3 = 0.5
```

这只是简化示例。真实训练会按照 Loss Mask、样本权重和训练配方决定哪些位置参与、怎样汇总。

当前 Batch Loss 表示：

> 当前参数在这一小批训练样本上的平均预测偏差。

它不是模型的最终能力分数，也不是全部训练数据上的精确表现。

## 第三步：许多 Batch 共同逼近整体目标

完整训练数据可能包含数十亿甚至更多 Token，不可能每更新一次参数就重新遍历全部数据。因此训练不断使用小批数据：

```text
Batch 1 → Loss → Gradient → 更新参数
Batch 2 → Loss → Gradient → 再更新参数
Batch 3 → 继续
……
```

每个 Batch 都是对整体数据的一次局部观察。随着模型见到大量不同 Batch，更新才逐渐反映整个目标数据分布。

这也是 **Stochastic Gradient（随机梯度）** 中“随机”的直观含义：当前 Batch 算出的方向只是整体方向的有噪声估计。

例如：

```text
Batch A 主要是中文百科 → 更强调当前中文样本
Batch B 主要是 Python 代码 → 更强调当前代码样本
```

所以一次更新可能让某些样本变好、另一些样本暂时变差；重要的是长期总体趋势。

## 把完整因果链串起来

```text
真实下一个 Token
→ 单个位置的 Cross-Entropy Loss
→ 汇总有效位置
→ 当前 Batch Loss
→ Backward 得到 Gradient
→ Optimizer 更新参数
→ 许多 Batch 与 Step
→ 逐渐降低整体训练目标
```

这里的“逐渐降低”不代表每个 Batch、每个样本或每一步都必须单调变好。

## 选读：两个评估概念

### Training Loss 与 Validation Loss

- **Training Loss**：在用于参数更新的数据上计算。
- **Validation Loss**：在不参与本轮更新的数据上计算，用来观察模型能否把规律迁移到未直接训练的样本。

如果 Training Loss 继续下降，而 Validation Loss 不再下降，模型可能越来越贴合训练数据，但泛化没有同步改善。

### Perplexity

Perplexity（PPL，困惑度）是由平均 Token Loss 转换得到的常见语言模型指标。通常 Loss 越低，PPL 也越低。

它衡量的是下一 Token 预测表现，不等于事实正确率或推理能力。不同 Tokenizer、数据集和上下文处理方式得到的 PPL 通常不能直接比较。

## 常见误解

### 当前 Batch Loss 就是模型的最终成绩

不是。它只反映当前参数在当前一小批数据和当前训练目标上的表现。

### 一个 Batch 的 Gradient 就是全体数据的精确方向

不是。它是基于局部样本得到的近似信号。

### Training Loss 下降就代表所有能力都在提升

不是。还要观察验证数据，以及事实、推理、安全等不同维度的评估。

## 理解检查

1. Token Loss、Batch Loss 和整体训练目标分别描述什么范围？
2. 为什么训练不在每次更新前遍历全部数据？
3. 为什么某一步更新可能不能改善每一个样本？

## 继续学习

- 上一篇：[[02-Loss与Cross-Entropy怎样衡量预测偏差|Loss 与 Cross-Entropy 怎样衡量预测偏差]]
- 下一部分：[[../03-Backward与参数更新/00-Backward与参数更新概览|Backward 与参数更新概览]]
