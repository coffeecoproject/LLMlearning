---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Token选择策略概览|Token选择策略概览]]"
previous: "[[01-Logits处理与Token选择|Logits处理与Token选择]]"
next: "[[03-Temperature怎样影响分布|Temperature怎样影响分布]]"
tags: [llm, generation, greedy, sampling, randomness]
---

# Greedy 与 Sampling

> [!summary]
> Greedy 每一步都选择当前分数最高的 Token；Sampling 则按照处理后的概率分布随机抽取一个 Token。

## 用同一分布对比

假设本轮候选概率是：

```text
“猫”  0.55
“狗”  0.30
“鸟”  0.15
```

- Greedy 一定选择“猫”；
- Sampling 更常选“猫”，但也可能选“狗”或“鸟”。

## Greedy 的特点

```text
每一步：argmax → 当前最高分 Token
```

优点是规则简单、相同计算条件下通常更稳定。局限是每一步的局部最高分不保证整段回答整体最佳，也可能产生僵硬或重复的路径。

## Sampling 的特点

Sampling 保留候选之间的概率差异，再进行随机抽取。它能带来更多样的措辞和路径，但也增加输出波动。Temperature、Top-k、Top-p 常用于先调整 Sampling 的候选分布。

## “随机”不等于“毫无规律”

高概率 Token 仍更容易被抽中。Sampling 的随机性发生在模型给出的条件分布之内，不是从全词表均匀乱选。

## 可复现性为什么仍不绝对

固定随机种子可以减少 Sampling 的随机差异，但不同硬件、并行方式、数值精度、Runtime 版本或服务端实现仍可能造成结果变化。托管 API 是否承诺完全复现，应以对应官方接口说明为准。

## 如何直观选择

- 需要格式稳定、便于比较时，可优先考虑确定性较强的策略；
- 需要创意和多样性时，可适度使用 Sampling；
- 正确性不能只靠把 Temperature 调低保证，还需要上下文、模型能力和验证机制。

## 理解检查

1. Sampling 为什么仍更常选择高概率 Token？
2. Greedy 每步最优为什么不保证整段全局最优？
3. 降低随机性为什么不等于消除幻觉？

下一篇：[[03-Temperature怎样影响分布|Temperature 怎样影响分布]]。
