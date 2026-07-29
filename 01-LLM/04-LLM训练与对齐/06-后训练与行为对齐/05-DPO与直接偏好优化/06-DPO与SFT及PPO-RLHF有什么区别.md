---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
previous: "[[05-一轮DPO训练怎样完整运行|一轮 DPO 训练怎样完整运行]]"
next: "[[07-DPO能改善什么又不能保证什么|DPO 能改善什么，又不能保证什么]]"
tags: [llm, dpo, sft, ppo, rlhf, comparison]
---

# DPO 与 SFT、PPO-RLHF 有什么区别

> [!summary] 一句话理解
> SFT 模仿一条参考回答，DPO 直接比较 Chosen 与 Rejected 的相对生成倾向，PPO-RLHF 则让当前 Policy 生成新 Rollout 并依据 Reward 进行受约束的策略更新。

## 先用一张表区分

| 维度 | SFT | 原始 DPO | 经典 PPO-RLHF |
|---|---|---|---|
| 主要数据 | Prompt + 参考回答 | Prompt + Chosen + Rejected | Prompt + Policy 新 Rollout |
| 核心信号 | 目标 Token | 候选之间的偏好关系 | Reward 与 Advantage |
| 独立 Reward Model | 不需要 | 不需要 | 通常需要 |
| Reference Model | 通常不需要 | 原始 DPO 通常需要 | 通常需要 |
| Value Model | 不需要 | 不需要 | 通常需要 |
| 每轮新 Rollout | 不需要 | 原始离线 DPO 不需要 | 需要 |
| 主要更新对象 | Language Model | Policy Model | Policy 与 Value 相关参数 |

三者都可能使用相似的 Transformer Forward、Backward 和 Optimizer，但训练目标不同。

## DPO 与 SFT：两个候选还是一个示范

### SFT

```text
Prompt → 参考回答
```

模型学习提高参考回答各个目标 Token 的概率。

### DPO

```text
Prompt
├── Chosen
└── Rejected
```

模型学习让 Chosen 相对 Rejected 更有利，并与 Reference 比较变化。

如果只把 Chosen 拿出来做 SFT，模型看不到：

```text
它当前容易生成的另一种回答
为什么相对不受希望
```

但这不表示 DPO 必然代替 SFT。现实路线常先用 SFT 建立基本助手行为，再用 DPO 做偏好优化。

## DPO 与 PPO-RLHF：固定比较还是新轨迹奖励

### DPO

```text
固定偏好对
→ 直接 Loss
→ 更新 Policy
```

### PPO-RLHF

```text
当前 Policy 生成新回答
→ Reward Model / 环境评分
→ Advantage
→ PPO 更新 Policy
```

PPO 能持续面对 Current Policy 新产生的行为，也更容易接入一般标量 Reward；代价是生成、评分和多模型协调更复杂。

DPO 工程更简洁、训练形态更接近离线微调；代价是受固定偏好数据覆盖范围约束，原始形式不会主动探索新输出。

## 哪一种更好

不能只按名称判断。选择取决于：

- 是否有高质量偏好对；
- 是否能获得可靠标量 Reward；
- 是否需要让当前 Policy 持续探索；
- 训练预算和基础设施；
- 是否需要程序可验证的环境反馈；
- 模型规模、任务类型和评估结果。

一个简化判断：

```text
已有稳定的成对偏好数据，重视训练简洁
→ DPO 可能合适

需要不断生成新轨迹并直接优化环境 Reward
→ 在线策略优化路线可能更合适
```

这只是选型方向，不是绝对规则。

## DPO 能否使用自动反馈

可以。Chosen / Rejected 不一定只能由人类产生，也可以来自：

- 规则；
- 测试结果；
- 数学答案验证；
- Model Judge；
- 多种反馈组合。

但如果主要反馈不是人类，就应准确称为偏好优化或 AI Feedback，而不要笼统描述成纯粹 RLHF。

## 三者训练后的运行方式相同吗

无论最终模型来自 SFT、DPO 还是 PPO-RLHF，普通 Decoder-only 模型运行仍是：

```text
上下文
→ Transformer Forward
→ Logits
→ Runtime 选择下一个 Token
→ 重复
```

训练方法改变 Weight，不会让部署模型自动携带训练 Loss、偏好对或 PPO 循环。

## 常见误解

### DPO 是 SFT 加入两个答案

不准确。它比较相对生成倾向并使用 Reference 约束，Loss 目标不同。

### PPO-RLHF 一定比 DPO 更高级

没有这种固定层级。两者是不同数据与工程条件下的训练路线。

### DPO 不能使用非人类反馈

可以，关键是形成可信的偏好关系；但反馈来源要如实标注。

## 理解检查

1. SFT 和 DPO 对一条样本分别需要几个候选回答？
2. 原始 DPO 与 PPO-RLHF 是否都需要每轮新 Rollout？
3. 为什么现实训练常先 SFT 再 DPO？
4. 三种训练完成后，普通运行为什么仍是下一 Token 生成？

## 继续学习

- 上一篇：[[05-一轮DPO训练怎样完整运行|一轮 DPO 训练怎样完整运行]]
- 下一篇：[[07-DPO能改善什么又不能保证什么|DPO 能改善什么，又不能保证什么]]
