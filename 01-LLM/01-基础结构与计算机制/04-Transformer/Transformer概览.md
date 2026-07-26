---
type: topic-index
module: 1
status: active
audience: non-specialist
parent: "[[01-LLM/01-基础结构与计算机制/基础结构与计算机制大纲]]"
previous: "[[Position位置机制概览]]"
next: "[[Output-Layer输出层概览]]"
tags: [llm, transformer, attention, ffn]
---

# Transformer

> [!goal]
> 理解 Transformer Block 怎样让各位置交换上下文信息、处理各自表示并稳定地逐层更新 Hidden State。

## Transformer 在主线中的位置

```text
Token ID
→ Embedding
→ Position
→ 多层 Transformer Block
→ Final Hidden States
→ Final Norm
```

Transformer 不是 Attention 的另一个名字。Attention 只是 Block 中负责位置间信息交换的一个子系统。

## 一个 Block 的非数学主线

```text
输入 Hidden States
→ Causal Self-Attention：各位置有选择地汇集上下文
→ Residual + Normalization：保留信息并稳定数据流
→ FFN / MLP：分别处理每个位置内部的特征
→ Residual + Normalization
→ 输出更新后的 Hidden States
```

不同模型可能采用 Pre-Norm、Post-Norm、并行子层等变体，因此上图表达职责关系，不假设所有实现的代码顺序完全一致。

## 阶段标注

> [!info] 两阶段共同
> Transformer Block 的 Attention、Residual、Normalization 和 FFN 前向计算，在 LLM 训练与运行阶段都会执行。训练阶段还会根据 Loss 计算梯度并更新参数；普通运行阶段使用固定参数，并可能增加 KV Cache、逐 Token 解码和服务调度等运行机制。

因此“某组权重是训练得到的”和“使用这组权重完成一次前向计算”是两件事，不能把整个 Transformer 误归为训练专属流程。

“两阶段共同”表示前向主干相同，不代表实现细节逐项完全一致。例如某些架构会在训练时启用 Dropout、在普通运行时关闭；不同阶段的 Batch 组织和内存策略也可能不同。

## 子结构与学习顺序

1. [[Transformer整体结构概览|Transformer 整体结构]]：先认识 Block 中有哪些组件以及数据怎样流动。
2. [[Causal-Self-Attention概览|Causal Self-Attention]]：理解上下文信息怎样被选择和汇集。
3. [[Residual与Normalization概览|Residual 与 Normalization]]：理解信息保留和数值稳定。
4. [[FFN概览|FFN / MLP]]：先理解 Dense FFN，再认识 MoE 怎样把同一职责改造成稀疏 Expert 路由。
5. [[Block堆叠与Hidden-State流动概览|Block 堆叠与 Hidden State 流动]]：把单层连接成完整模型主体。

## 简单形状示例

若一条序列有 3 个位置，`hidden_size=4`：

```text
输入 Hidden States：[3,4]
经过一个 Block：   [3,4]
经过多个 Block：   [3,4]
```

形状可以保持不变，但每层中的数值与所包含的上下文信息不断变化。

## 本专题边界

- 只讲静态结构和一次前向数据流；
- 不讲 Loss、梯度和参数更新；
- 不讲 KV Cache、请求 Batch 和逐 Token 生成循环；
- 不把 Attention Weight 当成完整思考解释。

## 基线与真实模型变体

必读主线先使用常见 Dense Decoder Transformer：

```text
Causal Attention
+ Dense FFN
+ Residual 与 Norm
```

真实模型可能替换其中某个子系统：

```text
Attention：Multi-Head Attention（MHA）/ Grouped Query Attention（GQA）/ Multi-Query Attention（MQA）/ Multi-head Latent Attention（MLA）/ 局部或滑动窗口变体
FFN：Dense FFN / Mixture of Experts（MoE，专家混合）
Norm：LayerNorm / RMSNorm，以及不同放置顺序
```

变体不会被当成另一套毫无关系的模型；阅读时始终先问“它替换了基线中的哪一块、保留了什么职责、改变了什么代价”。

## 当前进度

- [x] [[Transformer整体结构概览|Transformer 整体结构]]
- [/] [[Causal-Self-Attention概览|Causal Self-Attention]]
- [ ] [[Residual与Normalization概览|Residual 与 Normalization]]
- [ ] [[FFN概览|FFN / MLP]]
- [ ] [[Block堆叠与Hidden-State流动概览|Block 堆叠与 Hidden State 流动]]
