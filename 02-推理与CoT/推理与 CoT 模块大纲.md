---
type: module-outline
module: 2
status: planned
tags: [reasoning, cot, outline]
---

# 推理与 CoT 模块大纲

## 模块定位

研究模型如何表现出推理能力，以及如何利用中间步骤、搜索和验证提高复杂问题的正确率。

## 核心问题

> 模型生成一段看似有逻辑的过程，是否意味着它真的进行了可靠推理？

## 内容结构

1. 推理的不同定义：行为、过程与机制
2. Chain of Thought 的基本现象
3. 为什么中间步骤可能帮助模型
4. 外显 CoT 与内部计算的区别
5. CoT 的忠实性问题
6. Zero-shot CoT 与 Few-shot CoT
7. Self-Consistency
8. Tree of Thoughts、Graph of Thoughts 与搜索
9. 分解、规划与反思
10. Verifier 与可验证奖励
11. Test-time Compute
12. 推理模型及其训练方式
13. 数学、代码和形式化推理
14. 推理评测与常见失败

## 模块完成标准

- 不把“写出思维链”等同于“内部机制被完整展示”；
- 能解释 CoT 在什么条件下可能有效或失效；
- 能区分生成、搜索与验证；
- 能说明推理能力为什么需要进入系统级控制。
