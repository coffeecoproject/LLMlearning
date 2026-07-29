---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
previous: "[[04-Reference-Model与Beta分别做什么|Reference Model 与 Beta 分别做什么]]"
next: "[[06-DPO与SFT及PPO-RLHF有什么区别|DPO 与 SFT、PPO-RLHF 有什么区别]]"
tags: [llm, dpo, training-loop, policy-model, preference-data]
---

# 一轮 DPO 训练怎样完整运行

> [!summary] 一句话理解
> 一轮原始 DPO 训练从固定偏好数据中读取 Prompt、Chosen、Rejected，让 Current Policy 与 Frozen Reference 分别计算两条回答的生成倾向，再通过 DPO Loss 只更新 Policy。

## 训练开始前准备什么

通常需要：

1. 经过 SFT 的起点模型；
2. 一份可训练的 Current Policy；
3. 一份冻结的 Reference Model；
4. 配套 Tokenizer 与 Chat Template；
5. Prompt、Chosen、Rejected 偏好数据；
6. Beta、Learning Rate、Batch Size 等训练配置；
7. 独立验证集。

## 第一步：读取一批偏好对

```text
样本 1：Prompt + Chosen + Rejected
样本 2：Prompt + Chosen + Rejected
样本 3：Prompt + Chosen + Rejected
……
```

这些样本通常已经在训练前收集好，因此原始 DPO 常被描述为使用 Offline Preference Data。

这不表示偏好数据凭空产生；更早的数据阶段仍可能需要模型生成候选、人类比较和自动过滤。

## 第二步：整理输入并 Tokenize

```text
Prompt + Chosen
Prompt + Rejected
→ Chat Template
→ Tokenizer
→ Token IDs、Attention Mask、Response Mask
```

Prompt 为两条回答提供相同条件，Response Mask 确保序列倾向主要统计候选回答部分。

## 第三步：Current Policy 执行 Forward

```text
Policy(Prompt + Chosen)   → Chosen Token Log Probabilities
Policy(Prompt + Rejected) → Rejected Token Log Probabilities
```

训练系统累计回答 Token 的 Log Probability，得到 Policy 对两条完整回答的相对生成倾向。

## 第四步：Reference 执行或读取结果

```text
Reference(Prompt + Chosen)
Reference(Prompt + Rejected)
```

由于 Reference 冻结，它的结果可以现场 Forward，也可以预先计算后从数据中读取。

Reference 不执行 Backward，也不更新 Weight。

## 第五步：计算 DPO Loss

训练目标观察：

```text
Current Policy 相对 Reference
是否更偏向 Chosen 而不是 Rejected
```

如果 Current Policy 相对起点朝 Chosen 方向扩大了偏好差距，Loss 更满意；如果反向偏向 Rejected，Loss 会推动更明显的修正。

Beta 参与控制这个相对变化在 Loss 中的作用。

## 第六步：Backward 与更新 Policy

```text
DPO Loss
→ Backward
→ Gradient
→ Optimizer
→ 更新 Current Policy Weight
```

组件状态：

| 组件 | 是否更新 |
|---|---:|
| Current Policy | 是 |
| Reference Model | 否，冻结 |
| Tokenizer | 通常不更新 |
| 偏好数据 | 不会被 Backward 自动改写 |

DPO 没有独立 Reward Model、Value Model 或 PPO Old Policy 更新环。

## 第七步：处理下一批固定样本

```text
更新后的 Policy
→ 读取下一批偏好对
→ 再次计算四个序列倾向
→ 再次更新
```

与经典 On-policy PPO 不同，原始 DPO 不要求每次更新后立刻让新 Policy 生成新的 Rollout。

## 一个微型数字示例

以下为虚构教学数字：

```text
Reference：Chosen -5.0，Rejected -5.2，差距 0.2
Policy：   Chosen -5.1，Rejected -4.9，差距 -0.2
```

当前 Policy 反而更偏向 Rejected，DPO Loss 较不满意。

经过许多小幅更新后可能变成：

```text
Reference：Chosen -5.0，Rejected -5.2，差距 0.2
Policy：   Chosen -4.7，Rejected -5.5，差距 0.8
```

Policy 相对 Reference 更明显偏向 Chosen。真实训练同时处理大量样本，共享参数变化也会影响未见回答。

## 第八步：独立评估并部署

不能只看 DPO Loss。还应检查：

- 未见偏好对上的胜率；
- 事实与专业能力；
- 是否过度拒绝或迎合；
- 回答长度和风格变化；
- SFT 基础能力是否退化；
- 与人工及独立评审器的一致性。

训练完成后部署最终 Policy：

```text
用户请求
→ 最终 Policy Model
→ 普通自回归生成
```

Reference 和 DPO Loss 通常不进入运行阶段。

## 来源

> [!source]
> - [DPO 原始论文](https://arxiv.org/abs/2305.18290)将偏好优化转化为在参考策略约束下、直接以 Chosen / Rejected 训练语言模型的分类式目标。
> - 核对日期：2026-07-29。

## 常见误解

### Offline DPO 表示训练时完全不使用 GPU Forward

错误。离线只表示偏好数据预先准备好；Policy 和 Reference 仍需计算回答倾向。

### DPO 每批都会生成新的回答

原始离线 DPO 通常读取固定偏好对，不要求像 PPO 一样每轮生成 On-policy Rollout。

### 训练完成后必须同时部署 Policy 和 Reference

通常不需要。普通用户运行主要部署最终 Policy。

## 理解检查

1. 一轮 DPO 中哪四个回答倾向被使用？
2. 哪个模型更新，哪个模型冻结？
3. Offline Preference Data 与“不需要模型计算”有什么区别？
4. DPO 训练完成后，运行机制发生了什么变化？

## 继续学习

- 上一篇：[[04-Reference-Model与Beta分别做什么|Reference Model 与 Beta 分别做什么]]
- 下一篇：[[06-DPO与SFT及PPO-RLHF有什么区别|DPO 与 SFT、PPO-RLHF 有什么区别]]
