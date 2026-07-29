---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
previous: "[[01-RLHF到底是什么|RLHF 到底是什么]]"
next: "[[03-Reward怎样变成Policy-Model更新信号|Reward 怎样变成 Policy Model 更新信号]]"
tags: [llm, rlhf, rollout, trajectory, autoregressive-generation]
---

# Rollout 怎样产生训练经验

> [!summary] 一句话理解
> Rollout 是让当前 Policy Model 真正面对 Prompt 逐 Token 生成回答，并保存这次生成所需的概率与结果，供评分和后续参数更新使用。

## Rollout 是什么

**Rollout** 可以译为“展开”或“采样轨迹”。在语言模型 RLHF 中，它不是把已有标准答案读一遍，而是让当前模型实际生成新的回答。

```text
Prompt
→ Token 1
→ Token 2
→ Token 3
→ ……
→ EOS / 停止条件
→ 一条完整回答
```

这条从输入到结果的过程也常称为 **Trajectory（轨迹）**。

## 一个直白例子

Prompt：

```text
只用一句话解释 KV Cache。
```

Policy Model 可能逐步生成：

```text
KV → Cache → 保存 → 已计算 → 的 → Key → 和 → Value → ……
```

最后得到完整回答：

```text
KV Cache 保存已经处理过的 Token 所对应的 Key 和 Value，避免增量生成时反复计算历史部分。
```

训练系统会把这次生成及其相关信息保存成一条 Rollout 经验。

## 为什么不能只使用旧的偏好答案

Policy Model 每次更新后，输出分布都会发生变化。它可能产生旧数据中从未出现的新回答或新问题。

```text
旧偏好数据
→ 描述旧模型常见候选

更新后的 Policy
→ 可能产生新的风格、错误和取巧方式
```

通过 Rollout 采集当前 Policy 的实际输出，训练信号才能持续跟上正在变化的模型。

这类“让当前策略生成数据，再继续更新当前策略”的方式常被称为 **On-policy（在策略）**训练。这里表示训练数据来自当前或非常接近当前的 Policy，不表示面向用户实时改参数。

## 一条 Rollout 不只保存最终文字

为了后续训练，系统通常还需要保存或重新计算：

- Prompt 和生成 Token IDs；
- 每一步选择了哪个 Token；
- 当时 Policy 对这些 Token 给出的概率或 Log Probability；
- 回答在哪个位置结束；
- Attention Mask 等有效位置标记；
- Reward Model 给出的分数；
- Reference Model 对相同 Token 路径的概率；
- 后续计算出的 Advantage 等训练量。

**Log Probability** 可以先理解为 Token 概率的一种便于训练计算的数值表示，不需要在主线学习对数公式。

## 为什么生成需要一定探索

如果每次都只取概率最高 Token，模型可能反复生成非常相似的答案，难以发现其他可能获得更高 Reward 的表达。

训练 Rollout 常会使用一定程度的采样，让模型探索多个候选方向：

```text
大多数回答沿用当前较强倾向
+ 少量尝试其他合理路径
```

但随机性过强会产生大量无意义文本。因此 Temperature、采样范围和回答长度都需要控制。具体数值取决于训练方案，不存在一个通用固定设置。

## Rollout 与 Batch

训练不会通常只处理一个 Prompt，而会让 Policy 对一批 Prompt 生成回答：

```text
Prompt 1 → Rollout 1
Prompt 2 → Rollout 2
Prompt 3 → Rollout 3
……
```

这些回答长度可能不同，训练系统会通过 Padding、Mask 或动态调度组织计算。这里的 Batch 属于训练数据组织，不改变“每条回答仍按自回归方式逐 Token 产生”的事实。

## Rollout 生成完后会不会倒回去重写

不会。一次 Rollout 中已经采样出的 Token 构成固定训练记录。

后续发生的是：

```text
对完整轨迹评分
→ 根据结果更新 Policy Weight
→ 新 Policy 在下一批 Rollout 中可能生成不同回答
```

这不是在同一条已完成文本里反向擦掉 Token，而是用这次经验改变未来行为。

## 常见误解

### Rollout 是把整条回答一次性并行生成

不是。Decoder-only 模型仍按自回归方式生成；训练系统可以并行处理多条 Rollout，但每条内部存在 Token 先后依赖。

### On-policy 表示用户每次点赞都即时训练

不是。它表示训练样本主要由当前 Policy 生成，与面向用户的在线学习不是一个概念。

### 最终回答文字相同，训练记录就完全相同

不一定。生成时的模型版本、Token 概率、上下文和采样过程都可能不同。

## 理解检查

1. Rollout 与普通“读取一条标准答案”有什么区别？
2. 为什么 Policy 更新后需要继续生成新 Rollout？
3. 一条 Rollout 除了文本，为什么还要保留 Token 概率信息？
4. 已完成的 Rollout 会被反向重写吗？

## 继续学习

- 上一篇：[[01-RLHF到底是什么|RLHF 到底是什么]]
- 下一篇：[[03-Reward怎样变成Policy-Model更新信号|Reward 怎样变成 Policy Model 更新信号]]
