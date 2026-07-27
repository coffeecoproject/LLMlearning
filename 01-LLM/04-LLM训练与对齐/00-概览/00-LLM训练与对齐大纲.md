---
type: section-outline
module: 1
status: active
audience: non-specialist
parent: "[[01-LLM/00-概览/00-LLM 模块大纲|LLM 模块大纲]]"
previous: "[[00-性能指标与Runtime边界概览|性能指标与Runtime边界概览]]"
next: "[[01-LLM/05-能力形成与边界/00-能力形成与边界大纲|能力形成与边界大纲]]"
tags: [llm, training, alignment, outline]
---

# LLM 训练与对齐大纲

> 本部分研究模型参数和行为倾向怎样形成。只有在这里，才集中引入 Loss、梯度、反向传播和 Optimizer。

```text
原始数据
→ 清洗、过滤、去重与数据混合
→ Tokenize、切段与 Packing
→ 构造输入和错位一位的预测目标
→ Forward 与 Loss
→ Backward 与 Gradient
→ Optimizer 更新参数
→ Checkpoint
→ 预训练模型
→ 监督微调（SFT）、偏好优化、强化学习或蒸馏
→ 后训练模型
```

## 结构

1. [[00-训练的基本边界概览|训练的基本边界]]：训练、普通运行与 Tokenizer 训练的区别；
2. 数据工程：收集、许可、质量过滤、去重、数据混合与污染控制；
3. 样本构造：Tokenize、切段、文档边界、样本拼接（Packing）、补齐（Padding）与损失掩码（Loss Mask）；
4. Causal Language Modeling 目标：输入、错位一位的 Labels、教师强制（Teacher Forcing）与交叉熵（Cross-Entropy）；
5. Forward、Loss、Backward、Gradient 与 Optimizer 的因果链；
6. Step、Token Budget、Batch、Gradient Accumulation、Epoch、学习率与 Checkpoint；
7. 大规模预训练：混合精度、数据并行、张量并行、流水线并行与训练稳定性；
8. 持续预训练（Continued Pretraining）、监督微调（SFT）与指令学习：基础能力和行为格式怎样继续塑造；
9. 偏好数据、奖励模型（Reward Model）与人类反馈强化学习（RLHF）：反馈怎样成为训练信号；
10. 直接偏好优化（DPO）、强化学习与可验证奖励强化学习（RLVR）：不同后训练路线的共同点和区别；
11. Fine-tuning、低秩适配（LoRA）、参数高效微调（PEFT）、蒸馏与模型适配；
12. 数据偏差、评测污染、训练不稳定、灾难性遗忘与能力权衡。

## 边界

- Tokenizer 的词表训练与 LLM 参数训练必须区分。
- 普通聊天不会因为生成回答自动进入这里的参数更新流程。
- Agent 的长期学习循环属于后续“持续学习”模块。
- `Epoch` 是通用训练概念，但超大规模预训练常更关注已经处理的 Token 数、Step 和计算预算，不能把 Epoch 当作唯一进度单位。

## 真实训练路径校验

- Meta 对 Llama 3 的公开说明包含质量过滤、NSFW 过滤、语义去重和数据混合实验；
- InstructGPT 展示了 SFT、偏好排序数据、Reward Model 与强化学习路线；
- DPO 展示了不显式训练并调用独立 Reward Model 进行强化学习的直接偏好优化路线；
- DeepSeek-V3 官方报告包含预训练、SFT、强化学习、MoE 负载均衡和 Multi-Token Prediction 等真实变体。

来源：[Meta Llama 3 官方说明](https://ai.meta.com/blog/meta-llama-3/)、[InstructGPT 论文](https://arxiv.org/abs/2203.02155)、[DPO 论文](https://arxiv.org/abs/2305.18290)、[DeepSeek-V3 官方仓库](https://github.com/deepseek-ai/DeepSeek-V3)，核对日期：2026-07-27。
