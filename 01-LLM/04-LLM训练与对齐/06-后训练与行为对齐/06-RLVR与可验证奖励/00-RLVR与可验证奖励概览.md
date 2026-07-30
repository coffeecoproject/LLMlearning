---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-后训练与行为对齐概览|后训练与行为对齐概览]]"
previous: "[[00-DPO与直接偏好优化概览|DPO 与直接偏好优化概览]]"
tags: [llm, post-training, rlvr, verifier, reinforcement-learning, reasoning]
---

# RLVR 与可验证奖励概览

> [!summary]
> RLVR 让模型生成答案或执行轨迹，再由程序、测试或形式化规则判断结果是否满足明确条件，并把这个可验证结果作为强化学习 Reward 来更新模型。

## 所属阶段

本专题属于 **后训练阶段**。它讨论怎样用真实可检查结果训练模型，不是普通用户运行时的固定必经组件。

```text
训练 Prompt
→ Policy 生成一个或多个回答
→ Verifier 检查结果
→ 产生 Reward
→ 强化学习算法更新 Policy Weight
```

## 核心名词先认识

| 名称 | 直白理解 |
|---|---|
| RLVR | Reinforcement Learning with Verifiable Rewards，使用可验证奖励的强化学习 |
| Verifiable Reward | 能由明确程序、答案、测试或环境状态检查出来的奖励 |
| Verifier | 执行检查并返回结果的验证器，可以是规则、程序、测试框架或形式化证明器 |
| Ground Truth | 已知的参考事实或标准答案 |
| Rule-based Reward | 由显式规则产生的奖励，而不是神经网络凭偏好预测 |
| Outcome Reward | 根据最终结果是否正确产生的奖励 |
| Process Reward | 对中间步骤或过程状态提供的奖励 |
| Sparse Reward | 只有少数时刻得到反馈，例如整条回答结束后才知道对错 |
| Reward Hacking | 模型利用验证漏洞获得高分，却没有真正完成目标 |
| GRPO | 一种策略优化算法；可与可验证奖励结合，但不等于 RLVR 本身 |

## 八个学习问题

1. [[01-为什么需要可验证奖励|为什么需要可验证奖励]]：与人类偏好和 Reward Model 相比，它补上什么；
2. [[02-什么结果适合程序验证|什么结果适合程序验证]]：数学、代码、格式和环境任务为什么容易程度不同；
3. [[03-模型回答怎样变成Verifier-Reward|模型回答怎样变成 Verifier Reward]]：看清答案提取、检查和奖励生成；
4. [[04-一轮RLVR训练怎样完整运行|一轮 RLVR 训练怎样完整运行]]：串联 Rollout、Verifier 和 Policy 更新；
5. [[05-GRPO怎样利用同题多条回答形成更新信号|GRPO 怎样利用同题多条回答形成更新信号]]：理解相对组内比较；
6. [[06-Outcome-Reward与Process-Reward有什么区别|Outcome Reward 与 Process Reward 有什么区别]]：区分最终答案正确和过程可靠；
7. [[07-DeepSeek-R1与OpenAI推理模型公开了什么|DeepSeek-R1 与 OpenAI 推理模型公开了什么]]：建立公开事实与未知信息边界；
8. [[08-RLVR能改善什么又不能保证什么|RLVR 能改善什么，又不能保证什么]]：理解覆盖范围、Reward Hacking 与 Agent 验收边界。

## 一条主线

```text
题目：计算 17 × 8
→ Policy 生成推理和最终答案 136
→ Answer Extractor 提取 136
→ Verifier 与标准答案比较
→ 正确：Reward = 1
→ 强化学习算法提高这类生成轨迹的相对倾向
```

以上 `0/1` 是最简单示例。真实系统也可以组合正确性、格式、测试覆盖或环境状态等多个可验证信号。

## 三层不要混淆

| 层次 | 核心问题 | 例子 |
|---|---|---|
| Reward 来源 | 怎样判断结果好不好 | 单元测试是否通过 |
| 策略优化算法 | 怎样利用 Reward 更新模型 | PPO、GRPO |
| 普通运行系统 | 部署后怎样生成或执行 | Runtime、Agent、工具环境 |

所以：

```text
RLVR ≠ GRPO
Verifier ≠ Policy Model
训练时使用测试 ≠ 用户运行时必然重复训练
```

## 与 Reward Model 最直观的区别

```text
Reward Model：
从偏好数据中学习“这个回答看起来有多好”

Verifier：
按照明确规则检查“这个结果是否满足条件”
```

Verifier 通常在它覆盖的条件上更客观、更稳定，却只能处理可以明确写出检查方法的任务。

## 来源

> [!source]
> - [Tülu 3: Pushing Frontiers in Open Language Model Post-Training](https://arxiv.org/abs/2411.15124)，Ai2，2024：公开将 RLVR 作为 SFT、DPO 之后的后训练阶段，并使用数学答案和可验证指令约束产生奖励。
> - [DeepSeek-R1](https://arxiv.org/abs/2501.12948)，DeepSeek-AI，2025：公开描述准确性奖励、格式奖励与 GRPO 强化学习路线。
> - [DeepSeekMath](https://arxiv.org/abs/2402.03300)，DeepSeek-AI，2024：公开提出 GRPO。
> - 核对日期：2026-07-29。

## 学完后的理解标准

应当能够回答：

1. “可验证”指的是谁验证、验证什么？
2. 为什么数学答案和代码测试比开放写作更适合 RLVR？
3. Verifier 与 Reward Model 有什么根本差别？
4. GRPO 为什么不是可验证奖励本身？
5. 最终答案正确为什么仍不能证明中间推理全部正确？
