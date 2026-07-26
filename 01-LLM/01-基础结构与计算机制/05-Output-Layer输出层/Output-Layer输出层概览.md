---
type: topic-index
module: 1
status: planned
audience: non-specialist
parent: "[[01-LLM/01-基础结构与计算机制/基础结构与计算机制大纲]]"
previous: "[[Transformer概览]]"
tags: [llm, output-layer, lm-head, logits, softmax]
---

# Output Layer 输出层

> [!goal]
> 理解最后一层各位置的 Hidden State 怎样经过 Final Norm 与 LM Head 变成词表 Logits，并明确训练阶段与运行阶段怎样使用这些 Logits。

## 主线

```text
Final Hidden States [S, hidden_size]
→ Final Norm
→ LM Head
→ Logits [S, vocab_size]
```

之后分成两条阶段路线：

```text
训练阶段
→ 多个有效位置的 Logits 与错位一位的目标 Token 计算 Loss

运行阶段
→ 读取最后一个有效位置的 Logits
→ Logits Processor / Temperature / Softmax / 选择策略
→ 选出下一个 Token
```

## 计划子结构

1. Final Hidden States 与 Final Norm：输出层真正接收什么；
2. LM Head：怎样把 `hidden_size` 映射到 `vocab_size`；
3. Weight Tying：输入 Embedding 与输出 Head 是否共享参数；
4. Logits：每个位置怎样得到整套词表候选分数；
5. Softmax 与概率：它是数值转换，不是新的可学习权重层；
6. 所有位置与最后位置：训练和运行为什么使用不同切片；
7. Next Token Prediction 的准确含义与边界。

关键名词将分别建立原子笔记，不把它们挤在一篇长文中。

## 简单数字预览

假设玩具词表只有 4 个 Token：

```text
Logits：      [2.0, 1.0, 0.0, -1.0]
Softmax 后：  [0.64, 0.24, 0.09, 0.03]  （近似示意）
```

这里只需理解：在其他处理条件相同的情况下，Logit 越高，通常对应更高概率；概率总和为 1。指数函数和推导放在可选部分。

## 阶段边界

模型权重的前向主线到 Logits 为止。Softmax 作为必要概念保留，但要分别说明：训练 Loss 可能直接接收 Logits 并在内部完成等价计算；运行时还可能先修改 Logits，再形成概率并选择 Token。Temperature、Top-k、Top-p、采样、停止条件和逐 Token 循环属于普通运行与生成模块。

## 开放模型观察

DeepSeek-V3 官方权重说明把输出结构列为 `model.norm.weight` 与 `lm_head.weight`；Qwen3-8B 官方配置给出 `tie_word_embeddings=false`。这两项分别对应 Final Norm 和输入输出权重是否共享。

来源：[DeepSeek-V3 官方权重说明](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/README_WEIGHTS.md)、[Qwen3-8B 官方配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)，核对日期：2026-07-27。
