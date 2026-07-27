---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[FFN扩展结构概览]]"
previous: "[[FFN扩展结构概览]]"
next: "[[MoE基础与Dense-FFN对比]]"
tags: [llm, ffn, swiglu, geglu, gate]
---

# SwiGLU 与 GeGLU

> [!summary]
> SwiGLU 和 GeGLU 把基础 FFN 的单条中间路径改成两条：一条产生 Gate，一条产生候选特征；二者逐元素相乘后再降回 `hidden_size`。

## 从基础 FFN 到门控 FFN

基础教学结构：

```text
x → Up Projection → Activation → Down Projection
```

门控结构：

```text
              ┌→ Gate Projection → Activation ─┐
x ────────────┤                               × → Down Projection
              └→ Up Projection ────────────────┘
```

Gate 分支根据当前输入产生调节数值，Up 分支产生候选中间特征。两条分支宽度相同，才能对应维度相乘。

## 一个小数字直觉

教学示意：

```text
Gate分支：[0.1,0.8,0.0,0.5]
Up分支：  [2.0,1.0,3.0,-2.0]

逐元素相乘：
[0.2,0.8,0.0,-1.0]
```

真实 SiLU 或 GELU Gate 不局限在 0 到 1，不能理解成四个硬开关。

## SwiGLU 与 GeGLU 差在哪里

二者属于 GLU（Gated Linear Unit，门控线性单元）家族：

```text
SwiGLU → Gate 分支通常使用 Swish / SiLU
GeGLU  → Gate 分支使用 GELU
```

当前阶段不要求学习激活函数推导。重点是“两条投影 → 门控相乘 → Down Projection”。

## 与 MoE Router 的区别

```text
SwiGLU Gate
→ 调节一个 FFN 内部的中间特征维度

MoE Router
→ 在多个 Expert FFN 中选择计算路径
```

二者都可能被译为“门控”，但结构层级不同。

## Qwen3-8B 观察

Qwen3MLP 的主线是：

```text
down_proj(
  SiLU(gate_proj(x)) × up_proj(x)
)
```

官方配置给出 `hidden_size=4096`、`intermediate_size=12288`、`hidden_act="silu"`。只有把配置与模型实现结合起来，才能确认 SiLU 在门控结构中的实际位置。

> [!source]
> 来源：[Qwen3-8B 官方配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)、[Qwen3MLP 实现](https://github.com/huggingface/transformers/blob/main/src/transformers/models/qwen3/modeling_qwen3.py)。核对日期：2026-07-27。

## 常见误解

- SwiGLU 不是 Attention 变体。
- Gate 不只输出 0 或 1。
- Gate 与 Up 分支通常都映射到 `intermediate_size`。
- SwiGLU Gate 不是 MoE Router。
- 只看 `hidden_act` 字段不足以确认完整前向结构。

下一篇：[[MoE基础与Dense-FFN对比|MoE 基础与 Dense FFN 对比]]。
