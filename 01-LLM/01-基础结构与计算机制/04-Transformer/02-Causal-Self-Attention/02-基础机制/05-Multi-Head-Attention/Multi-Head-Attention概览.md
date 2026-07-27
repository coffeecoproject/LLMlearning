---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Attention基础机制概览]]"
previous: "[[Value加权与Context-Mixing概览]]"
next: "[[MHA-GQA与MQA概览]]"
tags: [llm, multi-head-attention, attention-head, output-projection]
---

# Multi-Head Attention

> [!summary]
> Multi-Head Attention 让多个 Attention Head 从同一组输入 Hidden States 出发，各自形成 Q、K、V、Attention Weight 和 Context；随后把各 Head 的结果拼接，并通过 Output Projection 返回模型主表示空间。

## 从单头结果继续

上一节已经得到单个 Head 对每个位置产生的 Context Vector：

```text
一个接收位置的 Query
→ 与可见 Keys 比较
→ Attention Weights
→ 对 Values 加权求和
→ 单头 Context
```

如果模型只有一个 Head，那么一个接收位置在这一层中只有一套匹配标准和一组 Attention Weights。Multi-Head Attention 增加的是多条并行的信息选择与汇总路径：

```text
同一组输入 Hidden States
├── Head 1：Q₁、K₁ → Softmax Weight₁；Weight₁ × V₁ → Context₁
├── Head 2：Q₂、K₂ → Softmax Weight₂；Weight₂ × V₂ → Context₂
└── Head N：Qₙ、Kₙ → Softmax Weightₙ；Weightₙ × Vₙ → Contextₙ
                    ↓
                 Concat
                    ↓
          Output Projection（W_O）
                    ↓
            Attention 子层输出
```

这里真正增加的不是“多个答案”，也不只是把一个向量切成几段，而是**多套分别归一化的 Attention Weight 和 Value 汇总路径**。一个更宽的单 Head 通常仍只有一套来源 Weight；多个 Head 则能为同一接收位置并行形成多套 Weight，再把多个 Context 合成一个输出。

这里的 Head 不是预先命名好的“语法模块”或“事实模块”。训练只提供更新信号，各 Head 最终学到什么由数据、目标、参数更新和模型整体协作共同形成。

## 阅读顺序

1. [[Attention-Head是什么]]：明确一个 Head 到底包含什么。
2. [[为什么需要多个Head]]：理解单套 Weight 的限制和并行表示的价值。
3. [[QKV怎样组织成多个Head]]：看懂 Projection、Reshape 和 Head 维度。
4. [[每个Head怎样独立产生Context]]：把单头 Attention 流程复制到多条并行路径。
5. [[Head拼接与Output-Projection]]：理解多个结果怎样回到模型主表示空间。
6. [[Multi-Head-Attention完整流程]]：用一组简单形状串起整个数据流。

## 一个最小形状预览

以下数字只用于教学示意。

```text
sequence_length = 3
hidden_size = 8
num_heads = 2
head_dim = 4
```

常见的标准 MHA 组织方式是：

```text
输入 Hidden States       [3, 8]
Q / K / V 投影结果       各为 [3, 8]
按 Head 重新组织         各为 [2, 3, 4]
每个 Head 的 Context     [3, 4]
两个 Head 拼接           [3, 8]
Output Projection 后     [3, 8]
```

`2 × 4 = 8` 只是说明两个 Head 的输出宽度拼接后回到 8 维。真实实现可能显式记录 `head_dim`，也可能从配置推导；不要仅凭这个教学例子推断所有模型配置。

## 必须分清的四件事

| 对象 | 作用 |
|---|---|
| Attention Head | 一条独立的 Q/K/V 匹配和 Value 汇总路径 |
| Multi-Head | 多条 Head 路径并行计算 |
| Concat | 沿特征维度保留并排列各 Head 的结果 |
| Output Projection | 用可学习线性映射混合拼接结果，并输出 Attention 子层表示 |

Concat 不是求和，Output Projection 也不是 Softmax。Softmax 已经在每个 Head 内把 Scores 转成 Weights；Output Projection 位于所有 Head 的 Context 产生之后。

## 阶段边界

> [!info] 两阶段共同
> Multi-Head Attention 是模型前向计算的一部分，训练阶段和运行阶段都会使用。本专题只讲相同的前向结构，不展开 Loss、梯度、参数更新、KV Cache、并发 Batch 或服务调度。

本专题先建立**标准 MHA**心智模型。MHA、GQA、MQA 怎样共享 K/V Head，进入[[MHA-GQA与MQA概览|下一专题]]；MLA 怎样改变 K/V 表示与缓存路径，进入[[MLA与注意力变体概览|扩展专题]]。

## 模型观察边界

现实中的 Decoder-only LLM 不一定使用标准 MHA。阅读 Qwen、OpenAI `gpt-oss`、Llama 或其他模型配置时，不能看到 `num_attention_heads` 就直接断言所有 Q、K、V Head 数量相同，还要结合对应版本公开的 K/V Head 和架构字段判断。这项真实模型比较留到下一专题。

## 完成标准

完成后应当能够：

1. 说明 Head 不是 Token、层或独立模型；
2. 解释多个 Head 为什么会产生不同 Weights 和 Context；
3. 说明原始 Hidden State 不是简单平均切开后直接做 Attention；
4. 根据 `hidden_size=8、num_heads=2、head_dim=4` 推演主要形状；
5. 区分 Head 内的 Softmax、Head 之间的 Concat 和最后的 Output Projection；
6. 说清 MHA 输出与 Residual、Normalization、FFN 和最终 Logits 的边界。

下一专题：[[MHA-GQA与MQA概览|MHA、GQA 与 MQA]]。
