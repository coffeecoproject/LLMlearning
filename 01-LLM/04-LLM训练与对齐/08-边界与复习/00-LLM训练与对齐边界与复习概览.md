---
type: topic-overview
module: 1
status: planned
audience: non-specialist
parent: "[[00-LLM训练与对齐大纲|LLM训练与对齐大纲]]"
previous: "[[00-模型适配与能力迁移概览|模型适配与能力迁移概览]]"
next: "[[01-LLM/05-能力形成与边界/00-能力形成与边界大纲|能力形成与边界大纲]]"
tags: [llm, training, boundaries, review]
---

# LLM 训练与对齐边界与复习概览

> [!summary]
> 训练能改变模型的参数能力与行为倾向，但训练数据、优化目标、计算过程和评测证据都有局限，因此不能把“已训练”等同于“已可靠”。

## 本专题将复习与区分

1. 从原始数据到后训练模型的完整主线；
2. 模型参数、Optimizer 状态、Checkpoint 和运行上下文；
3. 训练知识、当前上下文、外部记忆与持续学习；
4. 数据偏差、隐私、版权、重复与评测污染；
5. 训练不稳定、过拟合、灾难性遗忘和能力权衡；
6. 对齐、有帮助、安全、事实正确和任务可验证性之间的区别；
7. 使用开放权重仓库和技术报告时的事实、推断与未知边界。

## 最终学习目标

完成本部分后，应能不依赖口号地解释：

> 为什么 Base Model、Instruct Model、Reasoning Model 和 Agentic Model 可以使用相似的 Transformer 主干，却因数据和训练目标不同而呈现不同的回答倾向与能力边界？
