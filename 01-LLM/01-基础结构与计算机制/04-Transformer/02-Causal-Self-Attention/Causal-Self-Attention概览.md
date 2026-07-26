---
type: topic-index
module: 1
status: active
audience: non-specialist
parent: "[[Transformer概览]]"
previous: "[[Transformer整体结构概览]]"
next: "[[Residual与Normalization概览]]"
tags: [llm, attention, self-attention, causal-attention]
---

# Causal Self-Attention

> [!goal]
> 理解一个 Token 位置怎样根据当前表示寻找可见上下文、计算关注权重并汇总信息，同时受到 Causal Mask 限制而不能读取未来位置。

## 与上一专题的连接

上一专题已经建立：

```text
Token ID
→ Token Embedding
→ 加入或注入位置信息
→ 每个位置拥有一个初始表示
```

但每个位置的初始表示仍然主要从自己的 Token 出发。要形成上下文相关的 [[Hidden-State是什么|Hidden State]]，模型需要让不同位置之间有条件地交换信息，这就是 Self-Attention 的核心任务。

## 完整计算链预览

```text
输入 Hidden States
→ 线性投影得到 Q、K、V
→ 对 Q、K 注入位置影响（若采用 RoPE 等方案）
→ Q 与 K 计算 Attention Score
→ Scaling
→ 加入 Score 位置偏置（若采用 ALiBi 等方案）
→ 加入 Causal Mask
→ Softmax
→ Attention Weight
→ 对 V 加权求和
→ 多个 Head 合并
→ Output Projection
→ Attention 子层输出
```

不要现在记公式。阅读顺序会先回答“为什么”，再逐步拆开每一步。

## 子结构与学习顺序

1. [[Self-Attention的目的与边界概览|Self-Attention 的目的与边界]]：先理解它解决什么问题，以及 `Self`、`Causal` 各自是什么意思。✓
2. [[QKV投影系统概览|Q、K、V 投影系统]]：理解同一 Hidden State 为什么要产生三种角色表示。
3. [[从匹配强弱到信息权重概览|从匹配强弱到信息权重]]：先理解匹配、屏蔽和相对份额，再按需阅读数学实现。
4. [[Value加权与Context-Mixing概览|Value 加权与 Context Mixing]]：理解 Weight 怎样缩放对应 Value，并为每个接收位置形成单头 Context Vector。✓
5. [[Multi-Head-Attention概览|Multi-Head Attention]]：理解多个 Head、拼接与 Output Projection。✓
6. [[MHA-GQA与MQA概览|MHA、GQA 与 MQA]]：比较现代模型中的头部共享方式。
7. [[MLA与注意力变体概览|MLA 与注意力变体]]：以 DeepSeek-V3 为观察对象，理解压缩表示等变体替换了标准注意力的哪部分。此节为扩展阅读。

## 知识结构

```text
Causal Self-Attention
├── 01 Self-Attention 的目的与边界
│   ├── 为什么需要位置之间的信息混合
│   └── Self、Attention 与 Causal 分别是什么
├── 02 Q、K、V 投影系统
│   ├── 为什么需要三种角色
│   ├── Query
│   ├── Key
│   └── Value
├── 03 从匹配强弱到信息权重
│   ├── QK 点积与 Score
│   ├── Scaling
│   ├── Causal Mask
│   ├── 为什么运行时仍然保持因果可见性
│   ├── Softmax
│   └── Attention Weight
├── 04 Value 加权与 Context Mixing
│   ├── 为什么 Weight 要作用于 Value
│   ├── Value 加权求和怎样计算
│   ├── Context Vector 是什么
│   └── 为什么每个位置得到不同 Context
├── 05 Multi-Head Attention
│   ├── Attention Head 是什么
│   ├── 为什么需要多个 Head
│   ├── QKV 怎样组织成多个 Head
│   ├── 每个 Head 怎样独立产生 Context
│   ├── Head 拼接与 Output Projection
│   └── Multi-Head Attention 完整流程
├── 06 MHA、GQA 与 MQA
└── 07 MLA 与注意力变体（扩展）
```

## 阶段边界

> [!info] 两阶段共同
> Causal Self-Attention 的前向计算在 LLM 训练和运行时都会发生。训练阶段需要防止完整训练序列中的未来答案泄漏；运行阶段需要保持同样的左到右依赖，逐 Token 解码时未来 Token 尚不存在，优化实现也可能不显式创建完整三角 Mask。

本专题讲解一次 Attention 前向计算的因果链：

- 不讲 Loss、梯度或参数更新；
- 不讲完整逐 Token 生成循环；
- 不讲 KV Cache、Paged Attention 或请求调度；
- 不把 Attention Weight 当作完整思考过程解释。

这些内容分别属于训练、普通运行和能力边界模块。在本专题中即使需要提到，也只用于标明概念边界，不展开相应过程。

## 当前进度

- [x] [[Self-Attention的目的与边界概览|Self-Attention 的目的与边界]]
- [x] [[QKV投影系统概览|Q、K、V 投影系统]]
- [x] [[从匹配强弱到信息权重概览|从匹配强弱到信息权重]]
- [x] [[Value加权与Context-Mixing概览|Value 加权与 Context Mixing]]
- [x] [[Multi-Head-Attention概览|Multi-Head Attention]]
- [ ] [[MHA-GQA与MQA概览|MHA、GQA 与 MQA]]
- [ ] [[MLA与注意力变体概览|MLA 与注意力变体（扩展）]]
