---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-后训练与行为对齐概览|后训练与行为对齐概览]]"
previous: "[[00-Reward-Model与偏好评分概览|Reward Model 与偏好评分概览]]"
next: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
tags: [llm, post-training, rlhf, ppo, policy-model, alignment]
---

# RLHF 与策略模型更新概览

> [!summary]
> 经典 RLHF 让语言模型先生成回答，再用从人类偏好学来的 Reward 评价结果，最后通过强化学习小幅调整模型参数，使高奖励行为在未来更容易生成。

## 所属阶段

本专题属于 **后训练阶段**。它不是普通用户提问时发生的在线思考过程，而是模型发布前的一类参数训练过程。

```text
偏好数据
→ Reward Model 学会评分
→ RLHF 更新 Policy Model（本专题）
→ 得到新的已对齐模型版本
→ 部署到运行阶段
```

## 先认识核心名词

| 名称 | 直白理解 |
|---|---|
| RLHF | Reinforcement Learning from Human Feedback，基于人类反馈的强化学习 |
| Policy Model | 负责逐 Token 生成回答、正在被训练的语言模型 |
| Rollout | Policy Model 针对一批 Prompt 实际生成的回答及相关训练记录 |
| Trajectory | 从 Prompt 到完整回答的一条生成轨迹；在文本任务里常包含生成的 Token 序列 |
| Reward Model | 读取 Prompt 与完整回答，输出偏好分数的模型 |
| Reward | 对生成结果的数值反馈，可由 RM 分数和其他约束共同形成 |
| Reference Model | 冻结的参考语言模型，用来限制 Policy 偏离原有能力与风格过远 |
| KL Penalty | 根据 Policy 与 Reference 的差异施加的偏移惩罚 |
| Advantage | 这次结果相对“原本预期”好多少或差多少的训练信号 |
| Value Model / Critic | 估计当前状态通常能获得多少回报，帮助计算 Advantage |
| PPO | 经典 RLHF 中常用的一种策略更新算法，重点是限制单次更新过猛 |

> [!note]
> RLHF 是一条训练路线，PPO 是这条路线可以采用的一种强化学习算法。两者不是同义词，也不是所有现代后训练都必须使用 PPO。

## 七个学习问题

1. [[01-RLHF到底是什么|RLHF 到底是什么]]：从 SFT、偏好数据和 Reward Model 接到策略更新；
2. [[02-Rollout怎样产生训练经验|Rollout 怎样产生训练经验]]：模型怎样生成用于训练的新回答；
3. [[03-Reward怎样变成Policy-Model更新信号|Reward 怎样变成 Policy Model 更新信号]]：理解 Reward、Value 与 Advantage 的角色；
4. [[04-Reference-Model与KL约束为什么存在|Reference Model 与 KL 约束为什么存在]]：为什么不能只追求 Reward 最大；
5. [[05-PPO怎样限制每次更新幅度|PPO 怎样限制每次更新幅度]]：用直觉理解“朝好方向走，但一次不要走太远”；
6. [[06-一轮经典RLHF训练怎样完整运行|一轮经典 RLHF 训练怎样完整运行]]：串联所有组件；
7. [[07-RLHF能改善什么又不能保证什么|RLHF 能改善什么，又不能保证什么]]：明确事实、能力、价值和 Agent 验证边界。

## 一条主线

```text
Prompt
→ Policy Model 逐 Token 生成回答（Rollout）
→ Reward Model 给完整回答评分
→ Reference Model 提供偏移约束
→ 训练系统计算 Reward / Advantage
→ PPO Loss
→ Backward
→ Optimizer 更新 Policy Model Weight
→ 更新后的 Policy 再生成下一批回答
```

需要牢牢记住：已经生成的回答不会被倒回去修改。参数更新影响的是后续 Rollout 和最终部署模型的生成倾向。

## 四个模型角色不要混淆

| 角色 | 是否生成回答 | 是否在主训练中更新 | 主要作用 |
|---|---:|---:|---|
| Policy Model | 是 | 是 | 学习产生更高质量回答 |
| Reward Model | 否，只评分 | 通常冻结 | 预测回答符合偏好的程度 |
| Reference Model | 否，只作比较 | 冻结 | 限制 Policy 偏移过远 |
| Value Model / Critic | 否，只估计回报 | 通常更新 | 帮助判断本次结果是否优于预期 |

具体实现可以共享模型骨干或合并某些计算，但概念角色仍应分开。

## 经典路线与其他路线的边界

本专题主要解释 InstructGPT 一类公开方案中的 **Reward Model + PPO** 路线。

后面还会学习：

- DPO：直接从 Chosen / Rejected 更新语言模型，不运行完整的在线 PPO 环；
- RLVR：使用测试、数学答案等可验证结果产生奖励；
- 其他策略优化算法：可以替代 PPO 的某些部分。

它们不是本节要同时讲完的内容。

## 来源

> [!source]
> - [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155)，Ouyang 等，2022：公开描述 SFT、Reward Model 和 PPO 三阶段训练流程。
> - [Proximal Policy Optimization Algorithms](https://arxiv.org/abs/1707.06347)，Schulman 等，2017：PPO 原始论文。
> - [Learning to summarize from human feedback](https://arxiv.org/abs/2009.01325)，Stiennon 等，2020：展示语言任务中偏好奖励与策略优化的完整路线。
> - 核对日期：2026-07-29。

## 学完后的理解标准

应当能够回答：

1. RLHF 更新的是哪个模型？
2. Rollout 为什么既是模型输出，也是训练经验？
3. Reward Score 为什么不能不受限制地最大化？
4. Reference Model、旧策略快照和 Value Model 分别做什么？
5. RLHF 训练完成后，普通生成机制是否仍然是下一 Token 预测？

## 继续学习

- 下一小节：[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化]]
