---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
next: "[[02-DPO的一条训练样本怎样进入模型|DPO 的一条训练样本怎样进入模型]]"
tags: [llm, dpo, rlhf, preference-optimization]
---

# 为什么会出现 DPO

> [!summary] 一句话理解
> DPO 的出发点是：既然已有 Chosen 与 Rejected，能否不再训练独立评分器和运行复杂 PPO 循环，而直接让语言模型提高对 Chosen 的相对倾向。

## 从经典 PPO-RLHF 回顾

前一节的主线是：

```text
偏好对
→ 训练 Reward Model
→ Policy 生成 Rollout
→ Reward Model 评分
→ Reference 与 Value 提供约束和预期
→ PPO 更新 Policy
```

这条路线可以利用当前 Policy 不断生成新回答，但训练系统需要同时协调多个模型和数据阶段。

## 经典路线复杂在哪里

### 需要单独训练 Reward Model

偏好数据先训练一个评分器。评分器本身可能有偏差，还要单独验证和维护。

### 需要不断生成 Rollout

Policy 更新后要继续生成新回答，训练与生成交替进行，计算成本较高。

### 需要 Value 与 Advantage

结果级 Reward 要经过回报估计和信用分配，才能形成相对稳定的策略更新信号。

### 需要稳定 PPO

训练还要协调 Old Policy、Clip、KL Penalty 和多个 Loss，超参数与工程实现较复杂。

这些不是说 PPO-RLHF 错了，而是说明它并不是利用偏好数据的唯一办法。

## DPO 提出的核心问题

已有一条偏好对：

```text
Prompt：一句话解释 Token。
Chosen：Token 是模型处理信息时使用的基本离散单位。
Rejected：Token 就是一个英文单词。
```

我们真正想改变的是：

```text
在这个 Prompt 下
模型更倾向生成 Chosen 一类回答
而不是 Rejected 一类回答
```

DPO 直接把这个相对目标写进语言模型的训练 Loss。

## “直接”不等于简单记忆 Chosen

DPO 不是安装：

```text
if Prompt == 这句话:
    return Chosen
```

它仍然：

```text
把文本转成 Token IDs
→ 使用 Transformer Forward
→ 计算回答 Token 的 Log Probability
→ 构造 DPO Loss
→ Backward
→ 更新共享 Weight
```

大量不同偏好对共同改变模型在新 Prompt 下的生成分布。

## “不训练 Reward Model”怎样理解

原始 DPO 不需要先得到一个独立的：

```text
Prompt + 回答 → Reward Score
```

评分模型。

DPO 论文把受 KL 约束的奖励最大化问题转换成了可以直接由 Policy、Reference 和偏好对计算的目标。可以把它理解为：偏好信号被直接吸收到 Policy 的训练目标里，而不是先显式存进另一个 Reward Model。

这也是论文标题中“语言模型隐式承担奖励比较”的含义，但 Policy 的主要部署角色仍然是生成回答。

## DPO 为什么更像普通监督训练管线

训练可以读取一批固定样本：

```text
Prompt + Chosen + Rejected
```

然后重复执行 Forward、Loss、Backward。它不要求每次参数更新之间都运行完整的环境采样和 Reward Model 评分，所以在工程形态上更接近普通的离线微调。

但它的 Loss 不是 SFT 的单答案模仿 Loss，因此不能说 DPO 就是换名字的 SFT。

## 它解决了什么，又没有解决什么

DPO 主要简化：

- 多模型协调；
- Reward Model 单独训练；
- PPO Rollout 与 Value 估计；
- 在线策略优化的工程复杂度。

它没有自动解决：

- 偏好数据是否正确；
- Chosen 是否包含事实错误；
- 固定数据是否覆盖新 Policy 的输出；
- 模型是否过度贴合长度或风格；
- 现实任务是否真正完成。

## 常见误解

### DPO 完全不需要强化学习思想

不准确。原始推导来自受 KL 约束的奖励最大化问题，只是训练实现不运行经典 PPO 循环。

### DPO 不需要 Reward Model，所以没有偏好偏差

错误。偏差仍存在于 Chosen / Rejected 标签、候选生成和数据分布中。

### DPO 比 PPO-RLHF 永远更好

没有普遍结论。二者的数据使用方式、可扩展目标和工程代价不同，需要按任务评估。

## 理解检查

1. PPO-RLHF 为什么比普通离线微调更复杂？
2. DPO 的“直接”主要指直接利用什么更新什么？
3. 不训练独立 Reward Model 是否意味着不再需要 Loss 和 Backward？
4. DPO 简化工程后，哪些数据与真实性问题仍然存在？

## 继续学习

- 返回：[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]
- 下一篇：[[02-DPO的一条训练样本怎样进入模型|DPO 的一条训练样本怎样进入模型]]
