---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLVR与可验证奖励概览|RLVR 与可验证奖励概览]]"
previous: "[[05-GRPO怎样利用同题多条回答形成更新信号|GRPO 怎样利用同题多条回答形成更新信号]]"
next: "[[07-DeepSeek-R1与OpenAI推理模型公开了什么|DeepSeek-R1 与 OpenAI 推理模型公开了什么]]"
tags: [llm, rlvr, outcome-reward, process-reward, verifier, reasoning]
---

# Outcome Reward 与 Process Reward 有什么区别

> [!summary] 一句话理解
> Outcome Reward 只检查最终结果，Process Reward 则尝试评价中间步骤；前者更容易可靠验证，后者反馈更细，却更难定义和标注。

## Outcome Reward 是什么

**Outcome Reward（结果奖励）**只看任务结束后的结果：

```text
数学最终答案是否正确
代码最终是否通过测试
环境最终状态是否达标
```

例如：

```text
推理过程：若干步骤
最终答案：42
标准答案：42
→ Outcome Reward = 1
```

它不必逐句判断推理过程。

## Process Reward 是什么

**Process Reward（过程奖励）**尝试检查中间步骤：

```text
步骤 1 是否合理
步骤 2 是否使用正确公式
步骤 3 是否与前面一致
工具调用是否选择正确
失败后是否根据真实结果修正
```

它可以由：

- 人工逐步标注；
- Process Reward Model；
- 形式化证明器；
- 程序检查的中间状态；
- 工具环境返回结果。

产生。

## 一个“答案对但过程错”的例子

```text
题目：12 × 3
推理：12 + 12 = 24，再减去 12 得到 36。
最终答案：36
```

最终答案碰巧正确，但中间推理明显矛盾。

Outcome Verifier 只比较 `36`，可能给 Reward；Process Verifier 则有机会发现步骤错误。

这说明：

```text
结果正确
≠ 推理过程可靠
```

## 一个“过程合理但最终写错”的例子

```text
推理：12 + 12 + 12 = 36
最终答案：63
```

过程接近正确，但答案提取处写错。Outcome Reward 会判失败；Process Reward 可能认为前面步骤有价值。

## 两种奖励的主要权衡

| 维度 | Outcome Reward | Process Reward |
|---|---|---|
| 检查对象 | 最终结果 | 中间步骤 |
| 构造成本 | 通常较低 | 通常较高 |
| 客观性 | 有标准答案时较强 | 取决于步骤规则或标注者 |
| 信号密度 | 稀疏 | 更密集 |
| Credit Assignment | 较困难 | 更容易定位步骤 |
| 被表面过程欺骗 | 不评价过程 | 可能学习迎合过程评分器 |

## Process Reward 不一定是可验证奖励

如果过程由神经网络评分器判断：

```text
这一步看起来是否合理
```

它仍可能是学习得到的软评分，而不是程序可证明的 Verifiable Reward。

只有当中间步骤能由明确规则、证明器、代码执行或环境状态检查时，过程奖励才具有较强可验证性。

所以：

```text
Process Reward ≠ 自动等于 RLVR
Outcome Reward ≠ 自动一定可靠
```

## 为什么结果奖励仍能形成长推理

即使只有最终 Reward，Policy 也会探索不同 Token 轨迹。那些更容易到达正确结果的分解、检查和修正行为，经过许多 Rollout 后可能被间接强化。

但这种学习是间接的，可能同时强化：

- 真正有用的推理；
- 偶然猜对；
- 数据捷径；
- Verifier 漏洞；
- 不必要的冗长步骤。

因此需要难度设计、对抗评估和过程分析。

## Agent 任务中的结果与过程

### 结果验证

```text
最终测试是否全部通过
数据库是否达到目标状态
产物是否存在
```

### 过程验证

```text
是否读取了必要文件
是否使用真实 Tool Result
是否越权修改
失败后是否重新验证
```

对于高风险 Agent，只验证最终结果可能不够；某些危险操作即使最后达标，也不能被接受。

## 来源

> [!source]
> - [Training Verifiers to Solve Math Word Problems](https://arxiv.org/abs/2110.14168)，Cobbe 等，2021：展示生成多个解答并用 Verifier 选择结果的路线。
> - [Let's Verify Step by Step](https://arxiv.org/abs/2305.20050)，Lightman 等，2023：系统比较 Outcome Supervision 与 Process Supervision。
> - 核对日期：2026-07-29。

## 常见误解

### 最终答案正确说明每一步都正确

不成立，可能有错误抵消或偶然猜中。

### Process Reward 一定比 Outcome Reward 好

不一定。过程标注更昂贵，也可能把某种固定书写方式误当成正确推理。

### 推理写得越长，过程质量越高

没有必然关系。冗长过程可能包含更多错误和迎合评分器的内容。

## 理解检查

1. Outcome Reward 和 Process Reward 分别检查什么？
2. 为什么最终答案正确仍可能有错误推理？
3. Process Reward 在什么情况下才真正具有可验证性？
4. Agent 为什么有时需要同时验证结果和过程约束？

## 继续学习

- 上一篇：[[05-GRPO怎样利用同题多条回答形成更新信号|GRPO 怎样利用同题多条回答形成更新信号]]
- 下一篇：[[07-DeepSeek-R1与OpenAI推理模型公开了什么|DeepSeek-R1 与 OpenAI 推理模型公开了什么]]
