---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
next: "[[02-Rollout怎样产生训练经验|Rollout 怎样产生训练经验]]"
tags: [llm, rlhf, reinforcement-learning, policy-model]
---

# RLHF 到底是什么

> [!summary] 一句话理解
> RLHF 是先把人类对回答的偏好转成可计算的 Reward，再通过强化学习更新语言模型，让获得更好反馈的生成行为在以后更容易出现。

## 名称拆开理解

**RLHF** 全称是 **Reinforcement Learning from Human Feedback**：

- **Human Feedback**：人类对候选回答的比较、选择或评分；
- **Reinforcement Learning（强化学习）**：根据行动结果得到的 Reward，调整以后采取相似行动的倾向。

在语言模型中，可以先这样对应：

```text
环境给出的任务        → Prompt / 对话上下文
模型采取的行动        → 逐个生成 Token
一连串行动的结果      → 完整回答
结果反馈              → Reward
正在学习的策略        → Policy Model
```

这个对应是理解入口。真实训练还需要 Token 概率、约束、Value 估计和参数更新。

## 它从 SFT 接过什么

SFT 已经让模型学会：

- 把 User 消息当作要回应的指令；
- 使用对话格式；
- 模仿参考回答；
- 产生基本的帮助性和安全行为。

经典 RLHF 通常从 SFT Model 开始，而不是从随机参数或刚结束预训练的 Base Model 直接开始。

原因是强化学习需要在模型已经会正常生成回答的基础上，继续优化候选之间的细微质量差异。

## 为什么 SFT 后还需要 RLHF

SFT 的核心形式是：

```text
给定 Prompt
→ 模仿一条参考回答
```

但很多任务没有唯一理想答案。例如：

```text
A：正确但过长
B：简洁但漏掉关键条件
C：完整、清楚且符合用户长度要求
```

偏好数据可以说明 `C > A > B`，Reward Model 再把许多比较学成评分器。RLHF 的任务则是让生成回答的 Policy Model 真正朝这些高奖励结果调整。

## RLHF 不是把分数放进 Prompt

训练过程不是：

```text
System：请生成一个 Reward=10 的回答
```

而是：

```text
Policy 生成回答
→ 训练系统计算 Reward
→ Reward 参与 Loss
→ Backward 产生 Gradient
→ Optimizer 更新 Policy Weight
```

Reward 是训练信号，不是必须出现在上下文中的自然语言指令或 Special Token。

## RLHF 改变的仍然是 Token 概率

虽然评价对象是一整条回答，最终被更新的还是语言模型参数。

训练前，在相似 Prompt 下，模型可能更容易生成：

```text
冗长开场 → 模糊解释 → 自信但无依据的结论
```

经过许多 RLHF 更新后，相关 Token 路径的相对概率可能改变，使模型更倾向：

```text
直接回应 → 覆盖关键条件 → 说明不确定性
```

它不是保存一个“好回答按钮”，而是调整条件概率分布。

## 为什么称为“策略”模型

Policy 可以理解为模型面对当前上下文时，如何给下一 Token 分配概率。

```text
当前上下文
→ Policy Model 输出 Logits
→ 转成 Token 概率
→ 选择一个 Token
→ 新上下文
→ 再次选择
```

整条回答就是这一策略连续采取许多 Token 行动后的结果。

## RLHF 与普通运行的区别

### RLHF 训练阶段

```text
生成回答
→ 评分
→ 计算训练信号
→ Backward
→ 更新 Weight
```

### 用户运行阶段

```text
输入上下文
→ Forward
→ Logits
→ 生成 Token
```

普通运行通常没有 Backward，也不会因为一次回答得到的点赞立即修改模型 Weight。

## 常见误解

### RLHF 是模型在回答中反思自己

不是。反思文本属于模型生成内容；RLHF 是模型发布前的参数训练流程。

### RLHF 会给模型安装一个硬规则系统

不会。它主要改变行为概率，硬约束仍需要程序、权限和验证系统。

### 使用偏好数据训练都叫 PPO-RLHF

不准确。DPO 等方法也利用偏好数据，但不运行经典 Reward Model + 在线 PPO 的完整流程。

## 理解检查

1. RLHF 中的“行动”可以对应语言模型的什么？
2. 为什么经典 RLHF 通常从 SFT Model 开始？
3. Reward 是放进 Prompt 的文字，还是通过什么路径影响 Weight？
4. RLHF 训练与普通用户运行最关键的区别是什么？

## 继续学习

- 返回：[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]
- 下一篇：[[02-Rollout怎样产生训练经验|Rollout 怎样产生训练经验]]
