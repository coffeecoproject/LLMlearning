---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLVR与可验证奖励概览|RLVR 与可验证奖励概览]]"
previous: "[[04-一轮RLVR训练怎样完整运行|一轮 RLVR 训练怎样完整运行]]"
next: "[[06-Outcome-Reward与Process-Reward有什么区别|Outcome Reward 与 Process Reward 有什么区别]]"
tags: [llm, rlvr, grpo, group-relative, policy-optimization]
---

# GRPO 怎样利用同题多条回答形成更新信号

> [!summary] 一句话理解
> GRPO 让 Policy 对同一个 Prompt 生成一组回答，用组内 Reward 的相对高低判断哪些轨迹优于同题平均水平，从而避免为每个状态单独维护一个完整 Value Model。

## 先确认：GRPO 不等于 RLVR

```text
RLVR：Reward 从哪里来
→ 程序、测试或规则验证

GRPO：怎样利用 Reward 更新 Policy
→ 组内相对策略优化
```

GRPO 也可以使用非验证型 Reward；RLVR 也可以使用 PPO 等其他算法。DeepSeekMath 和 DeepSeek-R1 把 GRPO 与规则奖励结合，是一种具体组合。

## 为什么同一道题要生成多条回答

对于同一个 Prompt，Policy 可以采样：

```text
回答 A
回答 B
回答 C
回答 D
```

这些回答面对相同难度和条件，Reward 更容易做相对比较。

如果拿简单题的 `Reward=1` 和极难题的 `Reward=0` 直接比较，差异可能主要来自题目难度；组内比较能部分减少这种影响。

## 一个四候选示例

题目：

```text
15 × 6 等于多少？
```

Policy 生成四条 Rollout：

```text
A：90 → Reward 1
B：80 → Reward 0
C：90 → Reward 1
D：75 → Reward 0
```

组内平均 Reward：

```text
(1 + 0 + 1 + 0) ÷ 4 = 0.5
```

教学化相对信号：

```text
A、C：高于组内平均 → 正向
B、D：低于组内平均 → 负向
```

真实 GRPO 还会做标准化、策略比率、Clip 和 KL 约束等计算；主线不要求公式。

## 为什么可以不使用独立 Value Model

PPO 常用 Value Model 估计“当前状态通常能获得多少回报”。GRPO 改用同题一组回答的 Reward 统计作为相对基线：

```text
本题这组候选通常表现怎样
→ 当前候选高于还是低于组内水平
```

因此它可以省去独立 Critic / Value Model 的一部分成本。

“不使用独立 Value Model”不表示没有基线，也不表示训练不需要 Reference、Old Policy 或稳定性约束。

## 如果一组回答全部正确

```text
1，1，1，1
```

组内没有明显优劣差异，相对信号很弱。这类题可能对当前 Policy 太简单。

## 如果一组回答全部错误

```text
0，0，0，0
```

同样缺少区分，模型不知道哪条错误轨迹更接近正确。这类题可能太难，或者二元 Reward 太稀疏。

因此 RLVR 数据需要适当难度，也可能需要：

- 增加采样多样性；
- 更细粒度 Reward；
- 课程学习；
- 更好的起点模型；
- 过程级检查。

## GRPO 怎样影响 Token 倾向

获得正向相对信号的 Rollout，其所选 Token 路径会被适度提高倾向；负向轨迹则被适度降低。

```text
组内相对 Reward
→ Policy Loss
→ Backward
→ 共享 Weight 更新
```

它仍面临 Credit Assignment：最终答案正确并没有精确指出哪一个推理 Token 真正关键。

## 为什么仍要限制更新

一组候选只是有限样本。若一次把高 Reward 轨迹概率推得过高，模型可能：

- 失去多样性；
- 过度复制偶然成功的格式；
- 利用 Verifier 漏洞；
- 损伤其他任务能力。

所以 GRPO 实现通常仍包含策略更新幅度与相对 Reference 的约束。

## 来源

> [!source]
> - [DeepSeekMath](https://arxiv.org/abs/2402.03300)提出 GRPO，用组内 Reward 估计相对优势，并减少 PPO 中 Critic Model 的资源成本。
> - [DeepSeek-R1](https://arxiv.org/abs/2501.12948)公开使用 GRPO 进行大规模强化学习，并结合准确性与格式奖励。
> - 核对日期：2026-07-29。

## 常见误解

### GRPO 就是一次输出多个 Token

不是。它是同一 Prompt 采样多条完整 Rollout，再进行组内比较。

### 不用 Value Model 就等于没有相对基线

错误。组内 Reward 统计承担了相对基线角色。

### 只要增加候选数量，模型就一定变强

不一定。还受候选多样性、题目难度、Reward 质量和训练稳定性影响。

## 理解检查

1. RLVR 与 GRPO 分别回答什么问题？
2. 为什么同题候选适合做组内相对比较？
3. 全部正确或全部错误时，训练信号为什么变弱？
4. GRPO 省去 Value Model 后，为什么仍需要稳定性约束？

## 继续学习

- 上一篇：[[04-一轮RLVR训练怎样完整运行|一轮 RLVR 训练怎样完整运行]]
- 下一篇：[[06-Outcome-Reward与Process-Reward有什么区别|Outcome Reward 与 Process Reward 有什么区别]]
