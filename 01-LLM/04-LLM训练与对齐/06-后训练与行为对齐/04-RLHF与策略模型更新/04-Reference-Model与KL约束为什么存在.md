---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
previous: "[[03-Reward怎样变成Policy-Model更新信号|Reward 怎样变成 Policy Model 更新信号]]"
next: "[[05-PPO怎样限制每次更新幅度|PPO 怎样限制每次更新幅度]]"
tags: [llm, rlhf, reference-model, kl-divergence, reward-hacking]
---

# Reference Model 与 KL 约束为什么存在

> [!summary] 一句话理解
> Reference Model 是一份冻结的语言模型参照，KL 约束用它衡量当前 Policy 偏离原有分布的程度，从而防止模型为了追逐不完美的 Reward 而整体偏得太远。

## 为什么不能只让 Reward 越高越好

Reward Model 只是人类偏好的近似评分器，可能偏爱：

- 更长的回答；
- 更自信的语气；
- 固定格式或套话；
- 训练数据中的表面风格；
- 某些它自己也无法验证的结论。

如果 Policy Model 只追求 Reward 最大，可能学会利用这些漏洞：

```text
Reward 不断升高
但真实回答质量停止提升，甚至下降
```

因此训练系统还需要回答：

```text
新的 Policy 是否已经偏离原来的可靠语言能力太远？
```

## Reference Model 是什么

**Reference Model（参考模型）**通常是 RLHF 起点模型的一份冻结副本，例如 SFT Model 的副本。

```text
SFT Model
├── 复制为 Policy Model：继续训练
└── 复制为 Reference Model：冻结参数
```

面对相同上下文，训练系统比较二者对 Token 的概率分布：

```text
Policy：当前正在变化的生成倾向
Reference：训练开始时的生成倾向
```

Reference Model 不负责判断答案是否正确，也不生成标准答案；它只提供一个稳定的分布参照。

## KL 是什么

**KL Divergence（KL 散度）**是一种衡量两个概率分布差异的方法。

不需要学习公式，先看一个教学示例。某一步有三个候选 Token：

```text
Reference：A 0.50，B 0.30，C 0.20
Policy：   A 0.48，B 0.32，C 0.20
```

两者很接近，KL 差异较小。

如果 Policy 变成：

```text
Policy：   A 0.05，B 0.05，C 0.90
```

它与 Reference 的分布差异明显变大。

真实模型对整张 Vocabulary 计算相关差异，并沿生成序列累计；上面的三个 Token 只用于建立直觉。

## KL Penalty 怎样进入 Reward

训练系统通常会根据差异施加 **KL Penalty（KL 惩罚）**：

```text
回答获得较高 RM 分数
但 Policy 偏离 Reference 很远
→ 综合回报会被扣减
```

教学示例：

```text
Reward Model 分数：  1.6
加权 KL Penalty：   0.4
综合回报：           1.2
```

这里的 `0.4` 已假设经过训练系数加权，不代表 KL 与 Reward 天然使用同一单位。

## KL 约束在保护什么

它主要帮助维持：

- 基本语言流畅度；
- SFT 已学到的指令行为；
- 通用知识与能力；
- 输出分布的稳定性；
- 对 Reward Model 漏洞的抵抗力。

但它也会限制改变速度。如果约束过强，Policy 几乎不敢偏离 Reference，RLHF 就难以产生明显改善；如果过弱，又容易过度追逐 Reward。

所以这是一个平衡：

```text
学习新的偏好行为
↔ 保留原有能力与稳定性
```

## Reference Model 与旧策略快照不是同一个角色

后面学习 PPO 时还会遇到 **Old Policy（旧策略快照）**：

| 名称 | 比较目的 | 是否经常更新 |
|---|---|---:|
| Reference Model | 限制 Policy 相对训练起点偏移过远 | 通常长期冻结 |
| Old Policy Snapshot | 限制一轮 PPO 内的新 Policy 相对采样时策略变化过猛 | 每批或每轮更新 |

两者都与 Policy 做比较，但一个守住长期参照，一个控制单轮步幅。

## KL 约束仍然不是事实验证

Policy 与 Reference 很接近，只说明分布没有偏得太远，不说明：

- 回答事实正确；
- 代码测试通过；
- 用户目标完成；
- Reference 本身没有错误。

它是训练稳定器，不是正确性证明。

## 常见误解

### Reference Model 是标准答案模型

不是。它提供概率分布参照，不保证其回答正确。

### KL 越小越好

不一定。KL 永远接近零意味着 Policy 几乎没有学到新行为；目标是在改善与稳定之间取得平衡。

### KL 约束能彻底阻止 Reward Hacking

不能。它降低过度偏移风险，但无法修复 Reward Model 的全部漏洞。

## 理解检查

1. 为什么只最大化 Reward 可能让实际质量下降？
2. Reference Model 通常从哪里来，为什么冻结？
3. KL Penalty 试图限制什么？
4. Reference Model 与 Old Policy Snapshot 的目的有何不同？

## 继续学习

- 上一篇：[[03-Reward怎样变成Policy-Model更新信号|Reward 怎样变成 Policy Model 更新信号]]
- 下一篇：[[05-PPO怎样限制每次更新幅度|PPO 怎样限制每次更新幅度]]
