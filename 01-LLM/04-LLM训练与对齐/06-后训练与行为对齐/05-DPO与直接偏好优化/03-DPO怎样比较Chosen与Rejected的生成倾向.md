---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
previous: "[[02-DPO的一条训练样本怎样进入模型|DPO 的一条训练样本怎样进入模型]]"
next: "[[04-Reference-Model与Beta分别做什么|Reference Model 与 Beta 分别做什么]]"
tags: [llm, dpo, log-probability, preference-margin]
---

# DPO 怎样比较 Chosen 与 Rejected 的生成倾向

> [!summary] 一句话理解
> DPO 比较 Current Policy 相对 Reference Model 对 Chosen 与 Rejected 各自改变了多少，训练目标希望 Policy 比参考起点更明显地偏向 Chosen。

## 什么叫整条回答的生成倾向

语言模型逐 Token 输出概率：

```text
P(Token 1 | Prompt)
P(Token 2 | Prompt + Token 1)
P(Token 3 | 前面的全部内容)
……
```

训练系统可以把回答中每个实际 Token 的 **Log Probability（对数概率）**累积起来，形成整条 Response 的数值倾向。

不要求学习对数计算，只需知道：

```text
数值相对更高（通常更接近 0）
→ 模型相对更容易生成这条回答
```

因为概率小于 1，Log Probability 经常是负数；`-4` 高于 `-6`，不是分数写错了。

## 为什么不能只看 Chosen 的倾向

假设训练只提高 Chosen：

```text
Chosen 倾向提高了一点
Rejected 倾向提高得更多
```

最终模型反而更偏向 Rejected。

偏好训练真正关心的是两者差距：

```text
Chosen 倾向 - Rejected 倾向
```

这可以称为教学上的 **Preference Margin（偏好差距）**。

## 为什么还要与 Reference 比较

只要求 Policy 内部 `Chosen > Rejected`，可能让模型通过极端改变整套输出分布来完成目标。

原始 DPO 还比较 Policy 相对 Reference 的变化：

```text
Policy 对 Chosen 提高了多少
Policy 对 Rejected 提高或降低了多少
```

它希望 Policy 相对于训练起点更偏向 Chosen，同时保留参考约束。

## 一个四个数的教学示例

以下 Log Probability 是虚构示意，不是任何真实模型输出。

### Frozen Reference Model

```text
Chosen：  -5.0
Rejected：-5.2
偏好差距：0.2
```

Reference 只稍微偏向 Chosen。

### 训练中的 Current Policy

```text
Chosen：  -4.6
Rejected：-5.5
偏好差距：0.9
```

相对 Reference：

```text
Chosen 从 -5.0 提高到 -4.6：变化 +0.4
Rejected 从 -5.2 降低到 -5.5：变化 -0.3
```

所以 Current Policy 相对起点把偏好差距扩大了：

```text
0.9 - 0.2 = 0.7
```

DPO Loss 会认为方向符合标签。

## 如果 Policy 反而偏向 Rejected

```text
Policy Chosen：  -5.4
Policy Rejected：-4.8
```

Rejected 的倾向更高，且相对 Reference 朝错误方向移动。DPO Loss 会更明显地推动参数修正。

修正通过：

```text
Loss
→ Backward
→ Gradient
→ Optimizer
→ Policy Weight 改变
```

不是直接手工改写这四个序列分数。

## DPO 会怎样调整两条回答

目标是提高相对偏好差距，但参数更新不一定表现为：

```text
只提高 Chosen，Rejected 完全不动
```

它可能：

- 提高 Chosen；
- 降低 Rejected；
- 两者同时变化，但 Chosen 相对更有利；
- 通过共享参数影响其他相似回答。

因此 DPO 是相对分布优化，不是给每个答案分别写一个固定分数。

## 回答长度为什么可能影响数值

整条回答的 Log Probability 通常来自多个 Token 值的累积。回答越长，累积项越多，数值规模可能不同。

这不表示“DPO 必然喜欢短回答”或“必然喜欢长回答”，但数据长度偏差和具体实现的序列计算方式会影响训练，需要单独评估。后续 DPO 变体也会处理这类问题。

## 常见误解

### Log Probability 是 Reward Model 给出的分数

不是。它来自语言模型自身对回答 Token 的生成概率。

### DPO 只要让 Chosen 概率超过 Rejected 就完成

原始 DPO 还考虑相对 Reference 的变化，并在大量样本上训练和验证。

### 四个分数会被直接保存为模型规则

不会。它们用于计算本轮 Loss，最终影响共享 Weight。

## 理解检查

1. 为什么 `-4` 可以表示比 `-6` 更高的生成倾向？
2. 为什么不能只观察 Chosen，而不观察 Rejected？
3. 四个序列倾向分别来自哪两个模型和哪两条回答？
4. DPO 是否只能通过提高 Chosen 来扩大偏好差距？

## 继续学习

- 上一篇：[[02-DPO的一条训练样本怎样进入模型|DPO 的一条训练样本怎样进入模型]]
- 下一篇：[[04-Reference-Model与Beta分别做什么|Reference Model 与 Beta 分别做什么]]
