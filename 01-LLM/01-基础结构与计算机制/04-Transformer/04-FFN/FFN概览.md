---
type: subsection-index
module: 1
status: planned
audience: non-specialist
parent: "[[Transformer概览]]"
previous: "[[Residual与Normalization概览]]"
next: "[[Block堆叠与Hidden-State流动概览]]"
tags: [llm, ffn, mlp, swiglu, moe]
---

# FFN / MLP

> [!summary]
> FFN 在每个 Token 位置上独立使用同一组参数变换表示；Attention 负责位置之间交换信息，FFN 负责进一步处理每个位置已经汇集到的特征。

> [!info] 两阶段共同
> Dense FFN 或 MoE FFN 的前向路径在训练和运行时都会执行。训练阶段会更新 FFN、Router 与 Expert 参数，并可能加入负载均衡目标；运行阶段固定这些参数，但仍要为当前 Token 选择并执行相应计算路径。部署通信和请求调度属于运行模块。

## 计划结构

1. 为什么 Attention 后仍需要 FFN；
2. Dense FFN 的升维、非线性与降维；
3. 激活函数与 SwiGLU / GeGLU；
4. FFN 与 Attention 的职责区别；
5. Dense FFN 怎样扩展为 Mixture of Experts（MoE，专家混合）；
6. Router、Expert、Top-k 路由、总参数与激活参数；
7. MoE 的结构事实与训练负载均衡、部署通信成本之间的阶段边界。

数学只使用小维度形状示意，不展开激活函数推导。

> [!note] 学习顺序
> Dense FFN 是必读基线，MoE 是建立基线后的真实模型分支。MoE 不会被解释成多个完整 LLM 投票，而是同一 Transformer Block 中对 FFN 计算路径的稀疏选择。

## 开放模型观察计划

- Qwen3-8B 用于观察 Dense FFN 的 `intermediate_size` 与激活配置；
- DeepSeek-V3 用于观察 DeepSeekMoE、总参数和每 Token 激活参数的区别。

来源：[Qwen3-8B 官方配置](https://huggingface.co/Qwen/Qwen3-8B/blob/main/config.json)、[DeepSeek-V3 官方仓库](https://github.com/deepseek-ai/DeepSeek-V3)，核对日期：2026-07-27。
