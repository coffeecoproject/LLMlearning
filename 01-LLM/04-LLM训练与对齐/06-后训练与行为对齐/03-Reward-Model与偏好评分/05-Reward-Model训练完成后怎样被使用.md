---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Reward-Model与偏好评分概览|Reward Model 与偏好评分概览]]"
previous: "[[04-Ranking-Loss怎样让Chosen得分更高|Ranking Loss 怎样让 Chosen 得分更高]]"
next: "[[06-Reward-Model能判断什么又不能保证什么|Reward Model 能判断什么，又不能保证什么]]"
tags: [llm, reward-model, rlhf, policy-model, runtime]
---

# Reward Model 训练完成后怎样被使用

> [!summary] 一句话理解
> 在经典 RLHF 中，训练好的 Reward Model 会给 Policy Model 新生成的完整回答评分，训练算法再利用这个分数更新 Policy Model；它通常不是普通用户运行时每个 Token 都必须经过的组件。

## 先区分两个模型角色

### Reward Model

```text
读取：Prompt + 完整候选回答
输出：Reward Score
作用：评估回答与训练偏好的符合程度
```

### Policy Model

**Policy Model（策略模型）**是在强化学习阶段负责生成回答、并等待继续更新的语言模型。

```text
读取：Prompt / 上下文
输出：逐 Token 生成的回答
作用：实际作答
```

这里的“策略”可以直白理解为：面对输入时，模型倾向生成哪些 Token 的参数化行为方式。

## 经典 RLHF 中的一轮关系

```text
1. 给 Policy Model 一个 Prompt
2. Policy Model 逐 Token 生成一条完整回答
3. 把 Prompt + 回答交给 Reward Model
4. Reward Model 输出 Reward Score
5. RLHF 算法利用 Reward 和其他约束计算训练信号
6. 更新 Policy Model 的 Weight
7. 重复许多 Prompt
```

Reward Model 通常在这一步保持固定，承担“评分器”角色；被更新的是 Policy Model。

> [!note]
> 第 5 步怎样使用 PPO、怎样限制模型偏移，会在下一小节 `RLHF 与策略模型更新` 中单独学习。本篇只确认组件之间的数据流。

## 为什么要先生成完整回答再评分

经典偏好标签比较的是完整回答，因此 Reward Model 也常在完整回答生成后提供总体分数。

例如一个回答开头很好，但最后捏造事实：

```text
前半段清楚正确
→ 后半段出现虚构来源
```

只看第一个 Token 无法判断整体质量。

这也说明 Reward Model 的信号可能比较“稀疏”：它知道整条回答总体好或坏，却未必准确指出哪一个生成步骤出了问题。

## 它会不会参与普通用户运行

经典训练完成后，部署的 Chat Model 已把偏好训练造成的倾向保存在 Weight 中。普通运行通常是：

```text
用户上下文
→ 已对齐语言模型
→ Logits
→ Runtime 选择 Token
→ 完成回答
```

并不必然是：

```text
每生成一个 Token
→ Reward Model 审批一次
```

产品系统当然可以额外部署：

- 安全分类器；
- 回答重排器；
- Model Judge；
- 事实核验器；
- 代码测试与业务规则。

这些属于运行时系统设计，不能反过来假定“所有 RLHF 模型在线都带着 Reward Model”。

## 为什么不直接把 Reward Model 的高分答案返回用户

Reward Model 主要输出分数，不负责从零生成答案。系统若要在多个候选中重排，可以：

```text
语言模型生成多个候选
→ 评分器分别打分
→ 选择其中一个
```

但这是额外的推理策略，会增加计算量，也仍受评分器偏差限制。

## 与 SFT、偏好数据的完整串联

```text
预训练
→ 得到 Base Model

SFT
→ 用示范让模型学会基本指令行为

偏好数据
→ 记录同一 Prompt 下哪个候选更好

Reward Model 训练
→ 把许多比较学习成可重复使用的评分器

RLHF
→ 用评分器产生的 Reward 继续更新 Policy Model

部署运行
→ 用户使用训练完成的新模型版本
```

## 常见误解

### Reward Model 和 Policy Model 必须是同一个模型实例

不是。它们角色和输出不同；具体实现可以共享初始化来源，但训练后通常作为不同模型使用。

### Reward Model 会反向修改刚生成的 Token

不会。已经输出的 Token 不会被反向改写。训练算法利用分数更新参数，影响后续训练轮次中的生成倾向。

### 用户点赞会立刻经过 Reward Model 更新当前对话

通常不会。产品可以收集反馈供离线数据管线使用，但是否采纳、何时训练和何时发布新模型由具体系统决定。

## 理解检查

1. 在经典 RLHF 中，哪个模型生成回答，哪个模型给回答评分？
2. Reward Model 的分数怎样间接影响未来生成？
3. 为什么普通部署运行不一定在线调用 Reward Model？
4. 多候选重排为什么是额外系统设计，而不是所有模型的必经步骤？

## 继续学习

- 上一篇：[[04-Ranking-Loss怎样让Chosen得分更高|Ranking Loss 怎样让 Chosen 得分更高]]
- 下一篇：[[06-Reward-Model能判断什么又不能保证什么|Reward Model 能判断什么，又不能保证什么]]
