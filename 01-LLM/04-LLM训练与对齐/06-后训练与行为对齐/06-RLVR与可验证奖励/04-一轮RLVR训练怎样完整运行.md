---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLVR与可验证奖励概览|RLVR 与可验证奖励概览]]"
previous: "[[03-模型回答怎样变成Verifier-Reward|模型回答怎样变成 Verifier Reward]]"
next: "[[05-GRPO怎样利用同题多条回答形成更新信号|GRPO 怎样利用同题多条回答形成更新信号]]"
tags: [llm, rlvr, training-loop, rollout, verifier, policy-update]
---

# 一轮 RLVR 训练怎样完整运行

> [!summary] 一句话理解
> 一轮 RLVR 会让当前 Policy 对可验证任务生成 Rollout，由外部 Verifier 产生结果 Reward，再通过 PPO、GRPO 等策略算法更新 Policy，使更容易通过验证的轨迹在未来更可能出现。

## 训练开始前需要什么

1. 一个准备继续训练的 Policy Model；
2. 一批带有验证条件的 Prompt；
3. Answer Extractor 或动作解析器；
4. Verifier、测试或受控环境；
5. Reward 映射规则；
6. PPO、GRPO 等策略优化算法；
7. Reference Model 或其他偏移约束；
8. 独立评估集与防污染检查。

RLVR 定义的是 Reward 可验证，不规定只能使用哪一种策略算法。

## 第一步：采样任务

```text
数学题 + 标准答案
代码题 + 隐藏测试
格式任务 + Schema
环境任务 + 目标状态
```

任务难度需要与 Policy 匹配。若所有题都能轻易答对，训练信号不足；若全部失败，也很难知道哪个方向更好。

## 第二步：Policy 生成 Rollout

Policy 对每个 Prompt 逐 Token 生成一个或多个候选：

```text
Prompt
→ 推理 Token
→ 最终答案或动作
→ 完整 Rollout
```

训练系统保存生成 Token、当时概率、停止位置和必要 Mask。

这里仍是 Decoder-only 自回归生成，不是一次性并行写出整条推理。

## 第三步：提取待验证结果

```text
自然语言回答 → 提取最终答案
代码回答 → 提取代码块
工具任务 → 提取 Tool Call 或读取环境状态
```

解析失败需要单独标记，否则会把格式问题、模型能力问题和基础设施问题混在一起。

## 第四步：Verifier 执行检查

```text
数学：答案等价比较
代码：编译 + 隐藏测试
格式：Schema / 约束检查
环境：读取最终状态
```

Verifier 必须位于受控环境，并记录检查证据。

## 第五步：生成 Reward

最简单情况：

```text
正确 → 1
错误 → 0
```

也可以组合：

```text
正确性 Reward
+ 格式 Reward
- 安全违规 Penalty
- 相对 Reference 的偏移约束
= 策略训练使用的综合回报
```

Reward 设计过多、彼此冲突时，模型可能只优化最容易获得的部分。

## 第六步：计算相对更新信号

不同算法做法不同：

```text
PPO：结合 Value / Advantage、Old Policy 与 Clip
GRPO：比较同一 Prompt 下多条回答的组内相对 Reward
```

无论选择哪一种，目的都是把“哪些 Rollout 结果更好”转换成可执行 Backward 的 Loss。

## 第七步：Backward 与更新 Policy

```text
Policy Loss
→ Backward
→ Gradient
→ Optimizer
→ Policy Weight 更新
```

Verifier 一般不是通过 Backward 训练的神经网络组件。它运行规则并提供外部 Reward；被更新的是 Policy 及算法需要的相关参数。

## 第八步：新 Policy 再生成新 Rollout

```text
更新后的 Policy
→ 新一批题目与候选
→ 再次验证
→ 再次更新
```

这使训练可以发现固定偏好数据中没有的新解法，也会暴露新的 Reward Hacking 方式。

## 一个微型例子

题目：

```text
9 × 7 等于多少？
```

同一 Policy 生成：

```text
A：63  → Verifier 通过 → Reward 1
B：56  → Verifier 失败 → Reward 0
C：9+7=16 → Verifier 失败 → Reward 0
```

策略算法把 A 视为相对更有利的 Rollout，适度提高产生这类正确轨迹的倾向。

训练系统不是把 `9×7→63` 单独写进答案表，而是通过共享参数让相关计算和检查行为可能泛化到新题。

## 训练完成后的普通运行

最终部署的是更新后的 Policy Model。普通运行仍是：

```text
用户上下文
→ Policy Forward
→ Logits
→ Runtime 逐 Token 生成
```

训练时使用的 Verifier 不会自动随模型一起进入每个用户请求。是否在产品运行时继续验证，由 Agent 或应用架构决定。

## 常见误解

### RLVR 是一种固定的强化学习算法

不是。它描述 Reward 来源可验证，可以配合不同策略算法。

### Verifier 通过后会直接修改本次回答

不会。Reward 通过参数更新影响后续 Rollout；运行时是否重试是另一层 Agent 逻辑。

### 最终模型运行时必须同时加载训练 Verifier

不一定。Verifier 是否在线部署取决于产品设计。

## 理解检查

1. RLVR 为什么可以使用 PPO，也可以使用 GRPO？
2. Rollout 完成后到 Reward 之间还要经过哪些步骤？
3. Verifier 和 Policy 哪一个通常执行 Backward？
4. 为什么训练 Verifier 不会自动进入普通用户运行？

## 继续学习

- 上一篇：[[03-模型回答怎样变成Verifier-Reward|模型回答怎样变成 Verifier Reward]]
- 下一篇：[[05-GRPO怎样利用同题多条回答形成更新信号|GRPO 怎样利用同题多条回答形成更新信号]]
