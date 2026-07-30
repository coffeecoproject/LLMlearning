---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-RLVR与可验证奖励概览|RLVR 与可验证奖励概览]]"
previous: "[[06-Outcome-Reward与Process-Reward有什么区别|Outcome Reward 与 Process Reward 有什么区别]]"
next: "[[08-RLVR能改善什么又不能保证什么|RLVR 能改善什么，又不能保证什么]]"
tags: [llm, rlvr, deepseek-r1, openai-o1, evidence-boundary]
---

# DeepSeek-R1 与 OpenAI 推理模型公开了什么

> [!summary] 一句话理解
> DeepSeek-R1 技术报告公开了规则奖励、GRPO 和多阶段训练的具体主线；OpenAI 公开确认 o1 使用大规模强化学习学习推理，但没有公开到足以判断其具体 Verifier、Reward 组成或策略算法。

## 为什么要单独讨论证据边界

“推理模型使用强化学习”并不能自动推出：

```text
一定使用 GRPO
一定只使用可验证奖励
一定采用 DeepSeek-R1 相同的训练流程
一定在 Prompt 中加入固定推理 Token
```

不同机构公开的细节程度不同。理解模型时应把“官方明确说明”“根据公开资料推断”和“仍然未知”分开。

## 开放模型观察：DeepSeek-R1-Zero

DeepSeek 官方报告公开说明：

- DeepSeek-R1-Zero 以 DeepSeek-V3-Base 为起点；
- 在进入大规模 RL 前不先进行 SFT；
- 使用 GRPO 更新策略；
- 使用基于规则的 Accuracy Reward；
- 使用 Format Reward 约束思考与答案的输出格式；
- 没有在 R1-Zero 路线中使用神经网络形式的 Outcome 或 Process Reward Model。

报告中的准确性验证包括：

```text
数学题 → 判断最终答案是否正确
代码题 → 编译器或测试反馈
```

这是一条“规则奖励 + 策略强化学习”的公开实例。

## R1-Zero 出现了什么能力与问题

报告观察到训练过程中出现：

- 更长的 Chain of Thought；
- 自我检查；
- 反思和重新尝试；
- 复杂问题上的推理改善。

但 R1-Zero 也出现：

- 可读性较差；
- 多语言混杂；
- 重复等表达问题。

这说明只奖励结果正确，可能形成有效求解行为，却不自动得到适合用户阅读和通用对话的完整产品行为。

## DeepSeek-R1 为什么不是“纯 RLVR 一步完成”

正式的 DeepSeek-R1 使用了更完整的多阶段路线：

```text
Cold-start Data
→ 面向推理的 RL
→ Rejection Sampling 与 SFT 数据整理
→ 覆盖更多场景的 SFT
→ 兼顾推理与人类偏好的第二阶段 RL
```

官方概括为两次 RL 阶段和两次 SFT 阶段。

因此不能把最终 DeepSeek-R1 简化成：

```text
Base Model + 数学答案检查 = 完整 Chat Model
```

规则奖励、示范数据、通用任务和人类偏好各自解决不同问题。

## Format Reward 怎样理解

DeepSeek-R1-Zero 的 Format Reward 要求推理内容位于特定标签结构中，例如 `<think>...</think>`。

它验证的是：

```text
输出是否满足指定结构
```

而不是：

```text
标签内部每一步推理是否正确
```

格式可验证与推理可验证是两个层次。

## OpenAI o1 官方公开了什么

OpenAI 官方材料明确说明：

- o1 系列使用大规模强化学习进行复杂推理；
- 模型学习使用 Chain of Thought 改进策略、识别并修正错误；
- 表现随更多训练时计算和测试时思考计算提高。

这些信息支持：

```text
强化学习是 o1 推理能力形成的重要训练部分
```

## OpenAI 没有公开什么

截至核对日期，公开材料不足以确认：

- o1 或后续闭源推理模型是否采用原始 GRPO；
- 具体 Reward 是否属于哪一种 RLVR 配方；
- Verifier 的结构、数据和比例；
- 是否使用独立 Value Model；
- 训练阶段的完整 Loss 与超参数；
- 与 DeepSeek-R1 是否使用相同流程。

因此文档只能写：

```text
公开确认使用大规模 RL 学习推理
```

不能把 DeepSeek 的公开技术细节直接套到 OpenAI 模型上。

## 从 Codex 或 API 也无法反推出训练算法

客户端通常只能观察：

- 请求参数；
- 工具调用；
- 流式事件；
- Reasoning Effort 等产品控制；
- 最终输出与可见元数据。

这些属于运行接口，无法证明服务器端模型训练时使用了 GRPO、哪种 Reward 或多少条 RLVR 数据。

## 来源与事实状态

> [!source]
> - [DeepSeek-R1 官方仓库](https://github.com/deepseek-ai/DeepSeek-R1)与[技术报告](https://arxiv.org/abs/2501.12948)：本篇 DeepSeek 部分的主要事实来源。
> - [Learning to reason with LLMs](https://openai.com/index/learning-to-reason-with-llms/)与 [OpenAI o1 System Card](https://openai.com/index/openai-o1-system-card/)：确认 o1 使用大规模强化学习学习推理；未公开上述具体训练实现。
> - 核对日期：2026-07-29。

## 常见误解

### DeepSeek-R1-Zero 不使用 SFT，所以最终 R1 也完全不使用 SFT

错误。R1-Zero 与正式 R1 的训练路线不同，正式 R1 包含 Cold-start 和多阶段 SFT。

### 使用 Format Reward 等于验证每一步推理

不等于。它主要检查输出结构。

### OpenAI 说使用 RL，就能推导一定使用 GRPO

不能。策略算法和 Reward 细节没有充分公开。

## 理解检查

1. R1-Zero 与正式 R1 的训练路线有什么关键区别？
2. Format Reward 能验证什么，又不能验证什么？
3. OpenAI o1 官方明确公开了哪些训练层事实？
4. 为什么不能从 Codex 客户端源码推导模型采用 GRPO？

## 继续学习

- 上一篇：[[06-Outcome-Reward与Process-Reward有什么区别|Outcome Reward 与 Process Reward 有什么区别]]
- 下一篇：[[08-RLVR能改善什么又不能保证什么|RLVR 能改善什么，又不能保证什么]]
