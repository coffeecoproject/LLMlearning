---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[Block参数与边界概览]]"
previous: "[[Block参数与边界概览]]"
next: "[[Hidden-State在训练与运行中的边界]]"
tags: [llm, num-hidden-layers, parameters, qwen3, deepseek-v3, gpt-oss]
---

# num_hidden_layers 与每层参数

> [!summary]
> `num_hidden_layers` 通常表示 Transformer 主体堆叠的 Block 数量；每个 Block 通常拥有自己的 Attention、FFN 和 Norm 参数，所以增加层数通常也会增加模型参数与计算量。

## 配置字段怎样读

看到：

```json
{
  "num_hidden_layers": 36
}
```

入门阶段可以读成：

```text
Transformer 主体包含 36 个顺序堆叠的 Blocks
```

索引在代码中可能是 `0–35`，但数量仍是 36。

## 每个 Block 通常有哪些参数

```text
Attention
→ Q、K、V、输出投影等参数

FFN
→ Up、Gate、Down 或 Expert 等参数

Normalization
→ 取决于 LayerNorm、RMSNorm 等形式的可学习尺度参数
```

Residual 相加本身通常没有一张同类的大型权重矩阵。

## 不同 Block 是否共用参数

许多现代 Decoder-only LLM 的通常结构是：

```text
Block 1 使用第 1 套参数
Block 2 使用第 2 套参数
……
```

因此模型深度增加时，Attention 和 FFN 参数会按层重复出现，但每层数值不相同。

参数共享仍然是可能的架构选择。判断具体模型必须看对应版本的实现，不能只凭 Transformer 名称断言。

## 参数量与序列长度不同

模型有多少 Block 是结构配置。输入序列变长不会临时增加新的 Block，也不会增加模型参数量：

```text
参数量
→ 由模型结构和权重决定

本次计算量与中间状态规模
→ 会受到序列长度和 Batch 影响
```

## 三个开放模型观察

### Qwen3-8B

官方配置给出：

```text
num_hidden_layers = 36
hidden_size = 4096
intermediate_size = 12288
```

含义是 36 个 Transformer Blocks，主干宽度 4096；每个 Dense FFN 内部使用 12288 的中间宽度。

来源：[Qwen3-8B 官方配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)。

### DeepSeek-V3

官方推理配置给出：

```text
n_layers = 61
dim = 7168
```

这描述主 Transformer 堆叠的层数和主干宽度。模型还包含 MLA、DeepSeekMoE 和额外训练/预测结构，不能只凭 61 层概括完整架构。

来源：[DeepSeek-V3 官方 671B 推理配置](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/inference/configs/config_671B.json)。

### OpenAI gpt-oss-20b

官方开放配置给出：

```text
num_hidden_layers = 24
hidden_size = 2880
```

它提供了可检查的 OpenAI 开放模型示例。对于未公开内部配置的闭源 GPT 型号，不能根据 Codex CLI 或产品表现推断其 Block 数量。

来源：[OpenAI gpt-oss-20b 官方配置](https://huggingface.co/openai/gpt-oss-20b/blob/main/config.json)。

> [!source]
> 上述版本敏感配置核对日期：2026-07-27。

## 常见误解

- `num_hidden_layers` 通常不是统计所有 Linear Layer 的总数。
- 序列更长不会让模型临时多出新的参数层。
- `hidden_size`、`intermediate_size` 和 Block 数量分别描述宽度、FFN 内部宽度与深度。
- 闭源模型没有公布层数时，应记录为未知。

## 理解检查

1. `num_hidden_layers=36` 通常表示什么？
2. 为什么增加 Block 数量通常会增加模型参数？
3. 输入更长为什么增加计算，却不增加模型参数量？

下一篇：[[Hidden-State在训练与运行中的边界|Hidden State 在训练与运行中的边界]]。
