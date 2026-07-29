---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-后训练与行为对齐概览|后训练与行为对齐概览]]"
previous: "[[00-偏好数据与行为选择概览|偏好数据与行为选择概览]]"
next: "[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新概览]]"
tags: [llm, post-training, reward-model, preference-modeling, alignment]
---

# Reward Model 与偏好评分概览

> [!summary]
> Reward Model（奖励模型）把许多“回答 A 比回答 B 更好”的比较，学习成一个可以给新回答输出数值分数的模型；这个分数表示对训练偏好的预测，不是真理分数。

## 所属阶段

本专题属于 **后训练阶段**，位于两步之间：

```text
偏好数据构造
→ Reward Model 训练（本专题）
→ 使用 Reward 继续更新语言模型（后续 RLHF 专题）
```

普通用户向训练完成的 Chat Model 提问，属于 **运行阶段**。标准情况下，并不要求每次回答都在线调用 Reward Model。

## 核心名词先认识

| 名称 | 直白理解 |
|---|---|
| Reward（奖励） | 表示结果有多符合训练目标的数值信号，不是金钱或奖品 |
| Reward Model（RM） | 学习预测这种奖励分数的模型 |
| Reward Score | Reward Model 对一条候选回答输出的分数 |
| Scalar（标量） | 一个数；经典 Reward Model 通常为整条回答输出一个数 |
| Ranking（排序） | 比较多个回答时，关注谁的分数应当更高 |
| Ranking Loss（排序损失） | 衡量模型给出的分数顺序与偏好标签是否一致的训练误差 |
| Policy Model（策略模型） | 在 RLHF 中负责生成回答、等待继续训练的语言模型 |
| Reward Hacking | 模型学会钻评分规则的空子，拿到高分却没有真正完成目标 |

> [!note]
> “奖励”是训练系统中的数字信号；“奖励模型”则是预测这个信号的模型。两者不是同一个东西。

## 六个学习问题

1. [[01-为什么需要Reward-Model|为什么需要 Reward Model]]：为什么不能一直让人工给每个新回答评分；
2. [[02-Reward-Model的输入输出是什么|Reward Model 的输入输出是什么]]：看清它读取什么、输出什么；
3. [[03-偏好对怎样变成Reward-Model训练样本|偏好对怎样变成 Reward Model 训练样本]]：连接 Chosen、Rejected 与训练序列；
4. [[04-Ranking-Loss怎样让Chosen得分更高|Ranking Loss 怎样让 Chosen 得分更高]]：用简单数字理解参数更新目标；
5. [[05-Reward-Model训练完成后怎样被使用|Reward Model 训练完成后怎样被使用]]：区分 RM 训练、RLHF 训练和普通运行；
6. [[06-Reward-Model能判断什么又不能保证什么|Reward Model 能判断什么，又不能保证什么]]：理解评分器的偏差与边界。

## 一条完整主线

```text
同一 Prompt 下生成回答 A、B
→ 人类或其他反馈来源判断 A 更好
→ 组成 Chosen=A、Rejected=B 的偏好对
→ 同一个 Reward Model 分别读取 A、B
→ 输出两个 Reward Score
→ 如果 Chosen 分数没有更高，就产生较大的 Ranking Loss
→ Backward 与 Optimizer 更新 Reward Model 参数
→ 重复许多样本后，RM 学会预测相似场景中的偏好
```

训练结束后，Reward Model 可以面对未见过的新候选回答，估计哪个更符合它学到的偏好标准。

## 四层必须分开

| 层次 | 在做什么 | 是否更新参数 |
|---|---|---|
| 偏好数据构造 | 生成候选并比较谁更好 | 通常不更新 RM |
| Reward Model 训练 | 学习从回答预测偏好分数 | 更新 RM 参数 |
| RLHF 训练 | 用 RM 分数继续训练生成回答的语言模型 | 更新 Policy Model 参数 |
| 用户运行 | 训练完成的模型生成实际回答 | 通常不更新参数 |

后续的 PPO、KL 约束和 Policy Model 更新属于第三层，不在本专题展开。

## 最重要的边界

```text
Reward Score 高
≈ 更像训练反馈所偏好的回答
≠ 客观事实一定正确
≠ 现实任务已经完成
≠ 所有人都会喜欢
```

Reward Model 是偏好信号的近似器，也是一个会犯错的模型。

## 来源

> [!source]
> - [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155)，Ouyang 等，2022：公开描述“收集模型输出排序 → 训练 Reward Model → 用 PPO 优化策略模型”的经典流程。
> - [Learning to summarize from human feedback](https://arxiv.org/abs/2009.01325)，Stiennon 等，2020：展示用人类比较训练奖励模型，再把预测分数作为强化学习奖励。
> - 核对日期：2026-07-29。

## 学完后的理解标准

应当能够回答：

1. Reward Model 为什么不是一个固定评分规则？
2. 它通常读取什么、输出什么？
3. 为什么训练更关注 Chosen 与 Rejected 的相对顺序？
4. Reward Model 与生成回答的 Policy Model 是同一个角色吗？
5. 为什么高 Reward Score 不能直接证明回答正确？

## 继续学习

- 下一小节：[[00-RLHF与策略模型更新概览|RLHF 与策略模型更新]]
