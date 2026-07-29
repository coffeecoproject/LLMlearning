---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
previous: "[[05-PPO怎样限制每次更新幅度|PPO 怎样限制每次更新幅度]]"
next: "[[07-RLHF能改善什么又不能保证什么|RLHF 能改善什么，又不能保证什么]]"
tags: [llm, rlhf, ppo, training-loop, policy-update]
---

# 一轮经典 RLHF 训练怎样完整运行

> [!summary] 一句话理解
> 一轮经典 PPO-RLHF 会先让当前 Policy 生成回答，再分别取得偏好分数、参考分布和价值估计，构造受约束的更新信号，最后只对需要学习的模型执行 Backward 和参数更新。

## 开始前已经准备好什么

经典 InstructGPT 类路线进入 RLHF 前，通常已有：

1. 一批训练 Prompt；
2. 经过 SFT 的语言模型；
3. 由偏好对训练好的 Reward Model；
4. 一份冻结的 Reference Model；
5. 用于策略更新的 PPO 训练系统；
6. Value Model 或 Value Head。

可以从 SFT Model 复制出：

```text
Policy Model：继续更新
Reference Model：冻结
```

具体工程是否共享骨干、怎样分配显存不属于本节主线。

## 第一步：采样一批 Prompt

训练系统从 Prompt 数据中取一批任务：

```text
解释概念
总结文本
遵循格式
安全边界
代码问答
多轮对话
……
```

Prompt 分布会影响 Policy 最终在哪些场景得到改善。

## 第二步：Policy 生成 Rollout

Policy Model 对每个 Prompt 自回归生成回答：

```text
Prompt
→ Token 1
→ Token 2
→ ……
→ 完整回答
```

训练系统保存生成 Token、Mask，以及 Policy 在采样时给这些 Token 的概率信息。

## 第三步：Reward Model 给完整回答评分

```text
Prompt + 完整回答
→ Reward Model
→ Reward Score
```

Reward Model 通常冻结。它只提供偏好代理分数，不参与生成。

## 第四步：Reference Model 提供偏移信息

Reference Model 读取相同的 Prompt 和已生成 Token 路径，提供自己的概率分布。

训练系统比较：

```text
Policy 对这些 Token 的倾向
↔ Reference 对这些 Token 的倾向
```

差异过大时，通过 KL Penalty 降低综合回报。

## 第五步：Value Model 估计预期回报

Value Model 估计不同生成状态下通常能获得多少回报，训练系统再根据实际回报与预期计算 Advantage。

```text
实际表现优于预期 → 正 Advantage
实际表现差于预期 → 负 Advantage
```

这一步帮助减少仅由任务难度和随机采样造成的更新波动。

## 第六步：构造 PPO 训练目标

训练系统综合：

```text
Reward Model 分数
+/- 其他任务信号
- KL Penalty
→ 综合回报

综合回报 + Value 估计
→ Advantage

Advantage + Old Policy 概率
→ 受 Clip 限制的 PPO Loss
```

主线只需知道：训练目标既鼓励高回报行为，也限制单轮变化和长期偏移。

## 第七步：Backward 与参数更新

```text
PPO / Value 等 Loss
→ Backward
→ Gradient
→ Optimizer
→ 更新 Policy Model 与 Value 相关参数
```

典型角色状态：

| 组件 | 本轮主要状态 |
|---|---|
| Policy Model | 更新 |
| Value Model / Head | 通常更新 |
| Reward Model | 通常冻结 |
| Reference Model | 冻结 |

## 第八步：重新生成下一批 Rollout

参数更新后，Policy 已发生变化。训练系统继续取新 Prompt 或重新采样，生成下一批回答：

```text
新 Policy
→ 新 Rollout
→ 新 Reward
→ 新一轮更新
```

因此 RLHF 是一个生成与学习交替进行的循环，不是只对固定数据做一次 Forward。

## 用一个微型例子串起来

以下数值均为教学示意：

```text
Prompt：用一句话解释 Embedding。

Policy 回答：
Embedding 是把 Token 映射成模型内部连续向量的参数化表示。

RM 分数：              1.5
加权 KL Penalty：      0.3
综合回报：              1.2
Value 预期：            0.8
Advantage：            +0.4
```

PPO 因而适度提高这条生成路径的相对倾向，但 Clip 防止一次提高过猛。下一批遇到相似问题时，模型更可能生成清楚、简洁的定义。

它不会执行：

```text
把这句回答原样存入一个固定答案数据库
```

## 训练完成后怎样进入运行

训练结束会得到新的 Policy Model Weight。部署时通常只需要加载最终语言模型及 Runtime：

```text
用户请求
→ Tokenizer
→ 已对齐模型 Forward
→ Runtime 逐 Token 生成
```

Reward、Reference、Value 和 PPO 训练循环通常不进入每一次普通用户请求。

## 来源

> [!source]
> - [InstructGPT](https://arxiv.org/abs/2203.02155)公开描述先进行 SFT、再训练 Reward Model、最后使用 PPO 优化 SFT Policy 的三阶段流程，并使用相对 SFT 模型的 KL 约束。
> - [Learning to summarize from human feedback](https://arxiv.org/abs/2009.01325)给出了在人类偏好摘要任务中训练奖励模型和优化策略的实例。
> - 核对日期：2026-07-29。

## 常见误解

### 四个模型都在每一轮一起更新

不是。典型流程中 Reward Model 与 Reference Model 冻结，主要更新 Policy 与 Value 相关参数。

### RLHF 一轮就是用户的一轮对话

不是。这里的“一轮”是训练循环中的一个批次或更新周期，不等于产品对话轮次。

### 参数更新会改写刚才已经生成的回答

不会。它改变后续 Rollout 和最终部署模型的生成分布。

## 理解检查

1. 在 Policy 生成回答后，Reward、Reference 和 Value 分别提供什么？
2. 哪些模型典型情况下冻结，哪些参数会更新？
3. 为什么更新后要重新生成 Rollout？
4. 训练完成后，普通部署通常还需要在线加载哪些 RLHF 组件？

## 继续学习

- 上一篇：[[05-PPO怎样限制每次更新幅度|PPO 怎样限制每次更新幅度]]
- 下一篇：[[07-RLHF能改善什么又不能保证什么|RLHF 能改善什么，又不能保证什么]]
