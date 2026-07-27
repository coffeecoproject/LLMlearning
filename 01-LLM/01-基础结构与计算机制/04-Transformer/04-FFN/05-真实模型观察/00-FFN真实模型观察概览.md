---
type: reference-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN概览|FFN概览]]"
previous: "[[04-MoE总参数与每Token激活参数|MoE总参数与每Token激活参数]]"
next: "[[00-FFN边界与复习概览|FFN边界与复习概览]]"
tags: [llm, ffn, qwen3, deepseek-v3, gpt-oss]
---

# FFN 真实模型观察

> [!summary]
> 真实配置用于验证基础概念怎样落在具体模型上，而不是让初学者在理解 FFN 之前背诵模型字段。

## 阅读真实模型时问什么

```text
它使用 Dense FFN 还是 MoE？
主干 hidden_size 是多少？
中间 intermediate_size 是多少？
采用什么激活或门控？
MoE 有多少 Expert、每 Token 选择多少？
哪些是官方事实，哪些只是推断？
```

## Qwen3-8B：Dense 门控 FFN

官方配置：

```json
{
  "hidden_size": 4096,
  "intermediate_size": 12288,
  "hidden_act": "silu",
  "num_hidden_layers": 36
}
```

Qwen3MLP 实现：

```text
gate_proj：4096 → 12288
up_proj：  4096 → 12288
SiLU(gate) × up
down_proj：12288 → 4096
```

可以确认它是 Dense 门控 FFN，而不是 MoE Router 在多个 Expert 中做 Top-k。

来源：[Qwen3-8B 官方配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)、[Qwen3MLP 实现](https://github.com/huggingface/transformers/blob/main/src/transformers/models/qwen3/modeling_qwen3.py)。

## DeepSeek-V3：Dense 前层与 DeepSeekMoE

官方 671B 推理配置包含：

```json
{
  "dim": 7168,
  "inter_dim": 18432,
  "moe_inter_dim": 2048,
  "n_layers": 61,
  "n_dense_layers": 3,
  "n_routed_experts": 256,
  "n_shared_experts": 1,
  "n_activated_experts": 8
}
```

可以读出：

- 主干宽度为 7168；
- 前 3 层采用 Dense FFN；
- MoE 层包含 256 个 Routed Experts；
- 每 Token 激活 8 个 Routed Experts；
- 还有 1 个 Shared Expert；
- 单个 Expert 的中间宽度由 `moe_inter_dim=2048` 描述。

官方仓库给出的主模型统计约为 671B 总参数、每 Token 约 37B 激活参数。不能只使用 `8/256` 推算，因为还有 Attention、Shared Expert 和其他公共组件。

来源：[DeepSeek-V3 官方仓库](https://github.com/deepseek-ai/DeepSeek-V3)、[671B 配置](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/inference/configs/config_671B.json)、[官方推理实现](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/inference/model.py)。

## OpenAI gpt-oss-20b：可检查的 OpenAI MoE

官方模型卡和配置说明：

```text
residual stream dimension / hidden_size：2880
层数：24
Expert 数：32
每 Token 选择：Top-4
FFN 激活：门控 SwiGLU
总参数：20.9B
每 Token 激活参数：3.6B
```

gpt-oss 是 OpenAI 发布的开放权重模型，因此适合观察真实 MoE；它不能代表闭源 GPT-5.x 一定使用相同 FFN 架构。

来源：[OpenAI gpt-oss 官方模型卡](https://deploymentsafety.openai.com/gpt-oss/evaluation)、[`openai/gpt-oss-20b` 配置](https://huggingface.co/openai/gpt-oss-20b/blob/main/config.json)、[OpenAI gpt-oss 仓库](https://github.com/openai/gpt-oss)。

## 三者对比

| 模型 | FFN 路线 | 观察重点 |
|---|---|---|
| Qwen3-8B | Dense SwiGLU 类门控 FFN | `4096→12288→4096` 与三投影 |
| DeepSeek-V3 671B | Dense 前层 + DeepSeekMoE | Routed、Shared、Top-8 与总/激活参数 |
| OpenAI gpt-oss-20b | MoE | 32 Expert、Top-4、SwiGLU |

## 闭源模型怎样记录

对于 GPT-5.5、GPT-5.6 等闭源模型，如果官方未公布权重、配置或技术报告，应记录：

```text
内部 FFN / MoE 结构：未知
```

不能从 Codex CLI、模型名称、上下文窗口、Prompt Cache 或回答速度可靠反推 `intermediate_size`、Expert 数和激活函数。

## 字段速查

| 字段 | 通常表示什么 |
|---|---|
| `hidden_size` / `dim` | Block 主干宽度 |
| `intermediate_size` / `inter_dim` | Dense FFN 中间宽度 |
| `moe_inter_dim` | 单个 MoE Expert 的中间宽度 |
| `num_local_experts` / `n_routed_experts` | 可路由 Expert 数量 |
| `num_experts_per_tok` / `n_activated_experts` | 每 Token 选择数量 |
| `n_shared_experts` | Shared Expert 数量 |
| `hidden_act` | 激活函数线索，仍需结合实现 |

## 来源边界

以上公开配置核对日期：2026-07-27。精确版本事实可以记录；闭源结构未知时不能使用相邻模型或行为表现代替证据。

下一节：[[00-FFN边界与复习概览|FFN 边界与复习]]。
