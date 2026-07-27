---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[FFN参数与深入概览]]"
previous: "[[FFN输入输出形状与参数]]"
next: "[[FFN扩展结构概览]]"
tags: [llm, ffn, parameter-count, qwen3]
---

# FFN 参数量与规模

> [!summary]
> FFN 参数主要来自主干宽度与中间宽度之间的投影矩阵；中间宽度很大且每层都有 FFN，因此 FFN 常占 LLM 参数的重要部分。

## 先用小尺寸计算

设：

```text
hidden_size = H = 4
intermediate_size = I = 8
忽略 Bias
```

经典两投影 FFN：

```text
W_up：   4 × 8 = 32
W_down： 8 × 4 = 32
合计：64 个参数
```

三投影门控 FFN：

```text
W_gate：4 × 8 = 32
W_up：  4 × 8 = 32
W_down：8 × 4 = 32
合计：96 个参数
```

所以忽略 Bias 时：

```text
经典 FFN：约 2 × H × I
门控 FFN：约 3 × H × I
```

公式只用于看清参数来自哪里，不要求背诵。

## 为什么总量会很大

真实模型的 `H` 和 `I` 通常有几千，并且每个 Transformer Block 都有自己的 FFN 参数：

```text
单层 FFN 参数
× Transformer Block 数量
→ 全模型 FFN 参数
```

同一层的不同 Token 位置共享这些参数，所以序列变长不会增加模型参数量，只会增加这次前向计算的工作量。

## Qwen3-8B 实例

Qwen3-8B 官方配置给出：

```text
hidden_size = 4096
intermediate_size = 12288
num_hidden_layers = 36
```

其 Qwen3MLP 采用 `gate_proj`、`up_proj`、`down_proj` 三组主要投影。忽略很小的 Norm 参数和 Bias 差异，单层 FFN 主要参数约为：

```text
3 × 4096 × 12288
= 150,994,944
≈ 1.51 亿
```

36 层合计约：

```text
150,994,944 × 36
= 5,435,817,984
≈ 54.36 亿
```

按照同一配置对主要 Embedding、Attention、FFN 和 LM Head 矩阵做教学估算，模型约 81.9 亿参数，其中 FFN 约占 66%。这个比例用于展示“FFN 为什么是参数大户”，不应外推到所有 LLM。

> [!source] 开放模型观察
> 来源：[Qwen3-8B 官方配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)、[Qwen3MLP 实现](https://github.com/huggingface/transformers/blob/main/src/transformers/models/qwen3/modeling_qwen3.py)。核对日期：2026-07-27。占比是依据公开配置与主要矩阵计算得到的教学估算，不是官方发布的组件占比表。

## 参数占比为什么不能一概而论

不同模型会改变：

- `hidden_size` 与 `intermediate_size`；
- 两投影还是三投影门控；
- Dense FFN 还是 MoE；
- Expert 数量与每 Token 激活数量；
- Embedding 与 LM Head 是否共享；
- Attention、MTP 等其他组件规模。

因此不能说“所有 LLM 的 FFN 固定占三分之二”。必须指明精确模型版本和计算口径。

## 参数量、文件大小和计算量不是同一个数

```text
参数量
→ 模型包含多少可学习数值

权重文件大小
→ 参数量 × 每个参数的存储位数，加上格式开销

运行计算量
→ 本次前向实际执行哪些参数和多少 Token
```

Dense 模型三者联系较直接；MoE 会进一步把“总参数”和“每 Token 激活参数”分开。

## 理解检查

1. 为什么序列长度增加不会增加 FFN 参数量？
2. 三投影门控 FFN 为什么约有 `3HI` 个主要参数？
3. Qwen3-8B 的约 66% 为什么不能当成所有模型统一比例？
4. 参数量与权重文件大小有什么区别？

参数深入完成。扩展入口：[[FFN扩展结构概览|FFN 扩展结构概览]]。
