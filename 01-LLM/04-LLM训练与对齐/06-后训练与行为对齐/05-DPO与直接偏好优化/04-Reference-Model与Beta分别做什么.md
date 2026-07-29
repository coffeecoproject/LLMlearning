---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
previous: "[[03-DPO怎样比较Chosen与Rejected的生成倾向|DPO 怎样比较 Chosen 与 Rejected 的生成倾向]]"
next: "[[05-一轮DPO训练怎样完整运行|一轮 DPO 训练怎样完整运行]]"
tags: [llm, dpo, reference-model, beta, kl-constraint]
---

# Reference Model 与 Beta 分别做什么

> [!summary] 一句话理解
> 原始 DPO 用冻结的 Reference Model 记录训练起点的生成倾向，再通过 Beta 控制偏好优化与参考约束之间的关系，避免 Policy 为拟合有限偏好对而任意漂移。

## 为什么 DPO 仍需要 Reference Model

“直接偏好优化”容易让人误以为只需要 Current Policy：

```text
Chosen 概率提高
Rejected 概率降低
```

但如果没有稳定参照，Policy 可能为了少量偏好对大幅改变其他生成行为。

原始 DPO 通常从同一个 SFT Model 得到：

```text
Current Policy：继续训练
Reference Model：冻结
```

Reference 保存的是训练开始时模型对各种回答的相对生成倾向。

## Reference 提供的不是标准答案

对于每条偏好对，Reference Model 分别计算：

```text
Reference 对 Chosen 的生成倾向
Reference 对 Rejected 的生成倾向
```

训练系统再观察 Current Policy 相对这些起点发生了什么变化。

Reference 不负责：

- 判断 Chosen 是否事实正确；
- 输出 Reward Score；
- 替用户生成最终答案；
- 在普通运行时审批 Token。

它是概率分布参照，不是真理模型。

## 为什么 Reference 要冻结

如果 Reference 与 Policy 一起按相同目标不断变化，参照尺本身也在移动：

```text
Policy 向右移动
Reference 也向右移动
→ 难以知道 Policy 相对起点偏了多少
```

冻结后，它才能稳定表达 DPO 开始前的模型状态。

某些后续变体可以采用无参考或隐式参考方法，但那不改变原始 DPO 的主线。

## Beta 是什么

**Beta（β）**是 DPO 训练目标中的一个系数，用来调节偏好信号与相对 Reference 的约束关系。

直白地说，训练系统需要平衡：

```text
更明显地区分 Chosen / Rejected
↔ 不要无边界偏离 Reference
```

Beta 参与定义这个平衡和 Loss 对相对概率变化的敏感程度。

> [!warning]
> 不应只用“Beta 越大，模型一定越保守”这种一句话推断所有实现。论文推导、Loss 缩放、学习率和具体训练库会共同影响结果。对初学者最重要的是知道：Beta 是训练超参数，需要通过评估选择，不是模型能力分数。

## Beta 不是什么

它不是：

- 用户运行时的 Temperature；
- Reward Model 输出分数；
- Chosen 的人工评分；
- 模型参数规模；
- 一个写入 Prompt 的 Special Token。

它只在训练目标计算中起作用。

## 与 PPO-RLHF 的 KL 约束有什么关系

经典 PPO-RLHF 会显式计算相对 Reference 的 KL Penalty。原始 DPO 的推导同样来自受 KL 约束的奖励优化关系，但把它转换进了偏好 Loss。

因此可以从目标直觉上对应：

```text
PPO-RLHF：Reward + 显式策略优化 + KL 约束
DPO：偏好对 + Policy/Reference 相对倾向 + 直接 Loss
```

但不能把 DPO 的每一步计算机械等同于 PPO 的 Reward、Advantage 和 Clip。

## Reference 计算可以预先保存吗

因为 Reference Model 冻结，同一训练数据上的 Reference Log Probability 不会随 Policy 更新而变化。

工程上可以提前计算并保存这些数值，以减少重复 Forward；也可以训练时现场计算。这个选择影响存储与计算成本，不改变 DPO 的概念关系。

## 训练完成后还需要 Reference 吗

一般部署最终 Policy 时不需要：

```text
用户上下文
→ DPO 后的 Policy Model
→ Runtime 逐 Token 生成
```

Reference 主要服务训练约束，不会自动进入普通用户请求。

## 常见误解

### DPO 没有 Reward Model，所以完全没有参考约束

错误。原始 DPO 通常仍使用冻结 Reference Model。

### Reference Model 比 Policy 更正确

不一定。它只是训练起点，可能同样存在知识和行为错误。

### Beta 可以在用户请求中调节回答创造性

不是。运行时创造性通常与 Temperature、Top-p 等生成参数相关；DPO Beta 是训练参数。

## 理解检查

1. 原始 DPO 为什么保留 Reference Model？
2. Reference 为什么冻结？
3. Beta 主要调节哪两类目标之间的关系？
4. DPO 训练结束后，普通部署为什么通常不需要 Reference？

## 继续学习

- 上一篇：[[03-DPO怎样比较Chosen与Rejected的生成倾向|DPO 怎样比较 Chosen 与 Rejected 的生成倾向]]
- 下一篇：[[05-一轮DPO训练怎样完整运行|一轮 DPO 训练怎样完整运行]]
