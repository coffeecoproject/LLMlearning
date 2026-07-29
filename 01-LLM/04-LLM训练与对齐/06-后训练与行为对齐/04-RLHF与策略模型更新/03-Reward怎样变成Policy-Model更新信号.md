---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
previous: "[[02-Rollout怎样产生训练经验|Rollout 怎样产生训练经验]]"
next: "[[04-Reference-Model与KL约束为什么存在|Reference Model 与 KL 约束为什么存在]]"
tags: [llm, rlhf, reward, advantage, value-model, policy-gradient]
---

# Reward 怎样变成 Policy Model 更新信号

> [!summary] 一句话理解
> Reward 先说明完整回答总体表现如何，训练系统再结合偏移惩罚和“原本预期”计算 Advantage，最后提高有利 Token 路径的生成倾向、降低不利路径的倾向。

## Reward 不会自动修改参数

假设 Reward Model 给一条回答：

```text
Reward Score = 1.6
```

这个数字本身不会直接改变 Policy Model。仍需要：

```text
Reward
→ 构造强化学习训练目标
→ 计算 Policy Loss
→ Backward
→ Gradient
→ Optimizer 更新 Policy Weight
```

这与前面学过的参数更新主线相同，只是 Loss 的来源从“模仿参考 Token”变成“提高获得更好结果的生成倾向”。

## 为什么最终分数还不能直接使用

同样得到 `1.6`，在不同 Prompt 下意义可能不同：

```text
简单问题原本通常能得 2.0
→ 1.6 可能低于预期

高难问题原本通常只能得 0.5
→ 1.6 可能明显高于预期
```

训练更关心：这次结果相对于当前条件下的合理预期，是更好还是更差。

## Value Model 与 Advantage

**Value Model（价值模型，也称 Critic）**尝试估计：

```text
在当前 Prompt 和生成状态下
通常还能获得多少回报
```

**Advantage（优势）**则可以直白理解为：

```text
实际结果 - 原本预期
```

以下只是教学示例：

```text
经过约束后的实际回报：1.2
Value 预期：             0.7
Advantage：              +0.5
```

正 Advantage 表示这条生成路径比预期好，应适度提高其倾向。

```text
实际回报：0.2
Value 预期：0.7
Advantage：-0.5
```

负 Advantage 表示比预期差，应适度降低其倾向。

> [!note]
> `实际回报 - 预期` 是建立直觉的简化表达。真实 PPO 实现会考虑序列位置、折扣、逐 Token KL 信号和 Advantage 估计方法，本专题不要求推导公式。

## 一条总分怎样影响多个 Token

完整回答可能由几十或几百个 Token 组成，但 Reward Model 常只给一个总体分数。

训练系统会利用生成时记录的 Token 概率，把这次轨迹的 Advantage 关联到已经采取的 Token 行动：

```text
正 Advantage
→ 适度提高这条生成路径中所选 Token 的相对倾向

负 Advantage
→ 适度降低这条生成路径中所选 Token 的相对倾向
```

这里存在一个困难：总体分数并没有精确告诉系统“到底是哪一个 Token 做对或做错了”。这叫 **Credit Assignment（信用分配）问题**，即如何把最终结果归因到前面的多个行动。

## 为什么这比 SFT 的信号更间接

SFT 在每个 Assistant Token 位置都有参考 Token：

```text
这个位置应该更像目标 Token
```

经典结果级 RLHF 只知道：

```text
整条回答总体得分较高或较低
```

所以它能优化难以写成唯一标准答案的整体偏好，但训练信号更间接、噪声也可能更大。

## Reward 通常不只有 RM 分数

经典 RLHF 中，训练系统常把多个信号组合起来：

```text
Reward Model 分数
- 偏离 Reference Model 的惩罚
+ 可能的规则或任务奖励
= 用于策略训练的综合回报
```

下面的数字只为说明组合关系：

```text
RM 分数：       1.6
加权后的偏移惩罚：0.4
综合回报：      1.2
```

为什么需要偏移惩罚，将在下一篇单独解释。

## Value Model 也不是事实裁判

Value Model 只是在帮助估计“通常能得到多少训练回报”。它不检查：

- 答案是否经过权威来源验证；
- 代码是否真正运行；
- 文件是否真实写入；
- 现实任务是否完成。

它是降低强化学习更新波动的训练组件，不是 Agent 验收系统。

## 常见误解

### Reward 为正就一定提高所有 Token 的概率

训练使用的是相对预期、偏移约束和具体算法，不是只看分数正负的简单开关。

### Value Model 就是 Reward Model

不是。Reward Model 评价结果符合偏好的程度；Value Model 估计当前状态通常能获得多少回报。

### 一个低 Reward 能指出具体错字

不一定。结果级分数通常只说明整体较差，未必精确定位错误 Token。

## 理解检查

1. Reward Score 为什么不会自动修改 Policy Weight？
2. Advantage 试图表达“实际结果”相对什么更好或更差？
3. Reward Model 与 Value Model 的角色有何不同？
4. 为什么完整回答只有一个分数会产生 Credit Assignment 问题？

## 继续学习

- 上一篇：[[02-Rollout怎样产生训练经验|Rollout 怎样产生训练经验]]
- 下一篇：[[04-Reference-Model与KL约束为什么存在|Reference Model 与 KL 约束为什么存在]]
