---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-后训练与行为对齐概览|后训练与行为对齐概览]]"
previous: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
next: "[[00-RLVR与可验证奖励概览|RLVR 与可验证奖励概览]]"
tags: [llm, post-training, dpo, preference-optimization, alignment]
---

# DPO 与直接偏好优化概览

> [!summary]
> DPO 直接读取 Prompt、Chosen 和 Rejected，通过比较语言模型对两条回答的相对生成倾向来更新 Policy，不需要先训练独立 Reward Model，也不需要运行完整的 PPO-RLHF 循环。

## 所属阶段

本专题属于 **后训练阶段**，使用前面学过的偏好数据：

```text
Prompt
+ Chosen
+ Rejected
→ DPO Loss
→ Backward
→ 更新语言模型 Weight
```

这里的“直接”是相对于经典 `Reward Model → PPO` 路线而言，不表示跳过 Tokenizer、Forward、Loss、Backward 或参数训练。

## 核心名词先认识

| 名称 | 直白理解 |
|---|---|
| DPO | Direct Preference Optimization，直接偏好优化 |
| Policy Model | 正在根据偏好对更新的语言模型 |
| Reference Model | 冻结的参考语言模型，提供训练起点的生成倾向 |
| Chosen | 同一 Prompt 下相对更受偏好的回答 |
| Rejected | 同一 Prompt 下相对较差的回答 |
| Response Log Probability | 模型生成整条回答的相对倾向的一种训练数值表示 |
| Preference Margin | Chosen 相对 Rejected 的倾向差距 |
| Beta（β） | 原始 DPO 中调节偏好变化与参考约束关系的训练系数 |
| Offline Preference Data | 训练前已准备好的固定偏好数据，不要求每次更新时重新生成 Rollout |

## 七个学习问题

1. [[01-为什么会出现DPO|为什么会出现 DPO]]：经典 PPO-RLHF 的哪些复杂性推动了直接偏好优化；
2. [[02-DPO的一条训练样本怎样进入模型|DPO 的一条训练样本怎样进入模型]]：理解 Prompt、Chosen、Rejected 与 Token Mask；
3. [[03-DPO怎样比较Chosen与Rejected的生成倾向|DPO 怎样比较 Chosen 与 Rejected 的生成倾向]]：用简单数字理解四个序列分数；
4. [[04-Reference-Model与Beta分别做什么|Reference Model 与 Beta 分别做什么]]：为什么“直接”仍需要参考约束；
5. [[05-一轮DPO训练怎样完整运行|一轮 DPO 训练怎样完整运行]]：串联 Forward、Loss、Backward 与部署；
6. [[06-DPO与SFT及PPO-RLHF有什么区别|DPO 与 SFT、PPO-RLHF 有什么区别]]：按数据、组件和目标比较；
7. [[07-DPO能改善什么又不能保证什么|DPO 能改善什么，又不能保证什么]]：明确离线数据、事实和 Agent 验证边界。

## 一条主线

```text
同一个 Prompt
├── Chosen
└── Rejected

Current Policy 分别计算两条回答的生成倾向
Reference Model 分别计算两条回答的原始生成倾向
→ 判断 Current Policy 是否相对 Reference 更偏向 Chosen
→ 计算 DPO Loss
→ 只更新 Current Policy
```

## 与 PPO-RLHF 最直观的差别

### 经典 PPO-RLHF

```text
Policy 生成新 Rollout
→ Reward Model 评分
→ Value / Advantage
→ PPO 更新 Policy
```

### 原始 DPO

```text
读取固定偏好对
→ Policy 与 Reference 计算两条回答倾向
→ DPO Loss
→ 更新 Policy
```

DPO 省去了独立 Reward Model、Value Model 和在线 PPO Rollout 环，但仍需要 Reference Model 和语言模型训练计算。

## 本专题边界

本专题讲的是 Rafailov 等在 2023 年提出的原始 DPO 主线。后续出现了无参考模型、长度修正、多候选和在线 DPO 等变体，不在必修内容中展开。

## 来源

> [!source]
> - [Direct Preference Optimization: Your Language Model is Secretly a Reward Model](https://arxiv.org/abs/2305.18290)，Rafailov 等，2023：DPO 原始论文，提出不显式训练 Reward Model、直接使用偏好对优化语言模型的目标。
> - [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155)，Ouyang 等，2022：作为经典 Reward Model + PPO-RLHF 对照路线。
> - 核对日期：2026-07-29。

## 学完后的理解标准

应当能够回答：

1. DPO 的“直接”到底省略了什么，又没有省略什么？
2. 为什么原始 DPO 仍然需要 Reference Model？
3. DPO 比较的是两个完整答案的文字，还是模型生成它们的相对倾向？
4. DPO 与 SFT 的训练信号有什么不同？
5. 为什么 DPO 仍不能保证 Chosen 是正确答案？

## 继续学习

- 下一小节：[[00-RLVR与可验证奖励概览|RLVR 与可验证奖励]]
