---
type: section-outline
module: 1
section: 3
status: active
audience: non-specialist
parent: "[[01-LLM/LLM 模块大纲]]"
tags: [llm, training, alignment, outline]
---

# LLM 训练与对齐大纲

> 本部分研究模型参数和行为倾向怎样形成。只有在这里，才集中引入 Loss、梯度、反向传播和 Optimizer。

## 结构

1. [[3.1-训练的基本边界概览|训练的基本边界]]
2. 训练数据、样本与预测目标
3. 前向计算与 Loss
4. 反向传播、Gradient 与 Optimizer
5. Batch、Epoch 与学习率
6. 预训练
7. 监督微调与指令学习
8. 偏好学习与行为对齐
9. 蒸馏、微调与模型适配

## 边界

- Tokenizer 的词表训练与 LLM 参数训练必须区分。
- 普通聊天不会因为生成回答自动进入这里的参数更新流程。
- Agent 的长期学习循环属于后续“持续学习”模块。
