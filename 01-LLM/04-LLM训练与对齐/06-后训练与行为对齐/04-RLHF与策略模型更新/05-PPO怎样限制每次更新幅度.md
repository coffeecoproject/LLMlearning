---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
previous: "[[04-Reference-Model与KL约束为什么存在|Reference Model 与 KL 约束为什么存在]]"
next: "[[06-一轮经典RLHF训练怎样完整运行|一轮经典 RLHF 训练怎样完整运行]]"
tags: [llm, rlhf, ppo, clipping, old-policy]
---

# PPO 怎样限制每次更新幅度

> [!summary] 一句话理解
> PPO 利用当前 Rollout 判断哪些 Token 路径值得提高或降低，同时比较更新后的 Policy 与生成这批数据时的旧 Policy，避免一次参数更新把概率分布推得过远。

## PPO 是什么

**PPO** 全称是 **Proximal Policy Optimization（近端策略优化）**。

名称中的 `Proximal` 可以先理解为“保持在附近”：

```text
朝更高回报方向更新
但每次只走相对受控的一步
```

PPO 是强化学习算法，不是 Reward Model，也不是一种新的 Transformer 架构。

## 为什么强化学习更新容易过猛

假设一条回答获得较高 Advantage。最直接的想法是大幅提高其中所有已选 Token 的概率。

但语言模型参数被大量任务共享。一次过大更新可能导致：

- 其他 Prompt 的输出同时变化；
- 文本流畅度下降；
- 反复生成某些高分套话；
- 旧 Rollout 不再代表更新后的模型；
- 下一批训练突然不稳定。

因此“方向正确”还不够，还要控制步幅。

## Old Policy Snapshot 的作用

生成一批 Rollout 时，训练系统记录当时的 Policy，可把它理解为 **Old Policy Snapshot（旧策略快照）**。

```text
Old Policy
→ 生成这一批 Token
→ 保存当时选择这些 Token 的概率
```

准备更新时，再计算 Current Policy 对相同 Token 的概率，并比较两者变化。

```text
Old Policy：采样这些数据时的策略
Current Policy：经过本轮更新后正在形成的策略
```

## 一个不使用公式的例子

以下概率只是教学示意。

某个正 Advantage 的 Token 在 Old Policy 中概率为：

```text
0.20
```

更新后变成：

```text
0.24
```

这是一次相对温和的提高。

如果一次更新试图变成：

```text
0.80
```

说明新 Policy 已与生成这批 Rollout 的旧策略差异很大。继续相信同一批旧经验并猛烈更新，风险会明显上升。

## Clip 的直白理解

经典 PPO 使用 **Clipping（裁剪）**限制更新目标从过大概率变化中继续获得好处。

可以把它理解为：

```text
在合理范围内提高高 Advantage 行为
→ 训练目标鼓励

变化已经过大
→ 不再因为继续推远而获得同等训练收益
```

它不是简单把神经网络 Weight 截断到某个固定数，也不是把生成文本裁短。

## 正负 Advantage 怎样影响方向

### 正 Advantage

```text
这条路径比预期好
→ 适度提高所选 Token 路径的相对可能性
```

### 负 Advantage

```text
这条路径比预期差
→ 适度降低所选 Token 路径的相对可能性
```

PPO 同时关心方向和幅度：既要学，也不能让一批有限经验导致模型剧烈变化。

## PPO 与 KL 约束的差别

这两个限制经常同时出现，但参照不同：

```text
PPO Clip
→ 控制 Current Policy 相对 Old Policy 的单轮更新幅度

KL Penalty
→ 控制 Policy 相对冻结 Reference Model 的长期偏移
```

简化记忆：

```text
PPO Clip：这一步别迈太大
KL Penalty：整体别离起点太远
```

这只是直觉类比；真实机制作用在概率分布和训练目标上，并不是空间中的实际步伐。

## PPO 一般还会训练 Value Model

经典 PPO 系统通常还需要 Value Loss，让 Value Model 更准确地估计预期回报；也可能加入维持合理生成分布的其他 Loss。

因此一次训练的总目标并不只是：

```text
最大化 Reward Model 分数
```

而是多个目标共同维持学习效率和稳定性。

## 为什么现代方案不一定使用 PPO

PPO-RLHF 需要：

- 在线生成 Rollout；
- 运行 Reward Model；
- 运行 Reference Model；
- 维护 Value 估计；
- 协调多个模型和训练阶段。

计算和工程都较复杂。因此出现了 DPO 等直接偏好优化方法，以及其他策略优化算法。

这不表示 PPO 没有价值，而是说明：

```text
RLHF 是目标与训练路线
PPO 是其中一种实现选择
```

## 常见误解

### PPO 是让模型多思考几步

不是。它是训练阶段更新 Policy 参数的算法，与运行时 Reasoning Effort 不是同一个概念。

### Clip 会删除超出长度的 Token

不会。这里裁剪的是策略更新目标，不是文本长度。

### 有 PPO Clip 就不需要 Reference Model

不一定。二者控制不同时间尺度和不同参照下的偏移。

## 理解检查

1. PPO 为什么需要知道 Rollout 生成时的 Old Policy 概率？
2. Clip 控制的是文本长度还是策略更新幅度？
3. 正、负 Advantage 分别希望怎样改变生成倾向？
4. PPO Clip 与 KL Penalty 的参照对象有何不同？

## 继续学习

- 上一篇：[[04-Reference-Model与KL约束为什么存在|Reference Model 与 KL 约束为什么存在]]
- 下一篇：[[06-一轮经典RLHF训练怎样完整运行|一轮经典 RLHF 训练怎样完整运行]]
