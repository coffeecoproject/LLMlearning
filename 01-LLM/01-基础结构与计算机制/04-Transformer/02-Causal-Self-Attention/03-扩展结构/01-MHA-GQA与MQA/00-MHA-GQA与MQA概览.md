---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Attention扩展结构概览|Attention扩展结构概览]]"
previous: "[[00-Multi-Head-Attention概览|Multi-Head-Attention概览]]"
next: "[[00-MLA与注意力变体概览|MLA与注意力变体概览]]"
tags: [llm, mha, gqa, mqa, attention-variants]
---

# MHA、GQA 与 MQA

> [!summary]
> 标准 MHA、GQA 和 MQA 都保留多个 Query Head；核心差别是配备多少个 Key/Value Head：标准 MHA 一一对应，GQA 分组共享，MQA 全部共享。

## 从标准 Multi-Head Attention 继续

上一专题为了建立基线，把一个 Head 作为完整的 Q/K/V 路径：

```text
Head 1：Q₁、K₁、V₁
Head 2：Q₂、K₂、V₂
……
```

进入现代变体后，需要把它拆成两个数量：

```text
Query Head 数量
Key/Value Head 数量
```

原因是多个 Query Head 可以继续各自发起匹配，却不一定每个都拥有独立的 K、V Head。

## 先看一张最小关系图

假设有 8 个 Query Head：

```text
标准 MHA
Q0→KV0  Q1→KV1  Q2→KV2  Q3→KV3
Q4→KV4  Q5→KV5  Q6→KV6  Q7→KV7

GQA（示意为 2 个 KV Head）
Q0、Q1、Q2、Q3 → KV0
Q4、Q5、Q6、Q7 → KV1

MQA
Q0、Q1、Q2、Q3、Q4、Q5、Q6、Q7 → KV0
```

`KV0` 表示一组配套的 Key Head 和 Value Head，不表示只有一个 Key Token 或一个 Value Token。序列中每个位置仍会产生对应的 K、V 向量。

## 阅读顺序

1. [[01-为什么要区分Query-Head与KV-Head|为什么要区分Query-Head与KV-Head]]：先拆开以前被统称为 Head 的两类数量。
2. [[02-MHA怎样一一对应Q与KV-Head|MHA怎样一一对应Q与KV-Head]]：用标准 MHA 建立一一对应基线。
3. [[03-MQA怎样让所有Query-Head共享KV|MQA怎样让所有Query-Head共享KV]]：理解多个 Query 怎样共用一组 K/V。
4. [[04-GQA怎样让Query-Head分组共享KV|GQA怎样让Query-Head分组共享KV]]：理解介于两端之间的分组方式。
5. [[05-MHA-GQA与MQA完整对比|MHA-GQA与MQA完整对比]]：用同一组数字和形状横向比较。
6. [[06-怎样从模型配置判断Head结构|怎样从模型配置判断Head结构]]：读取真实配置字段，并观察 Qwen3 与 OpenAI `gpt-oss`。

## 三者共同保留什么

无论采用哪种方式：

- 每个 Query Head 仍可以拥有自己的 Query 表示；
- 每个 Query Head 仍会计算自己的 Score、Softmax Weight 和 Context；
- 所有 Head 仍遵守相同的 Causal 可见性约束；
- 各 Query Head 的 Context 最后仍需要组合并经过 Output Projection；
- 得到的仍是一个 Attention 子层输出，不是多个答案。

改变的是 K/V Head 的独立程度，不是把 Attention 的全部步骤删除。

## 阶段标注

> [!info] 两阶段共同
> MHA、GQA、MQA 都是模型 Attention 的前向结构，LLM 训练和运行时都会使用。训练阶段学习相应的 Q/K/V 投影参数；普通运行阶段读取固定参数并按同样的 Head 对应关系计算。

> [!note] 阶段边界
> 本节只解释 Head 怎样对应和共享。减少 KV Head 是 MQA/GQA 的重要运行效率动机，但 KV Cache 张量、显存占用、内存带宽和 Prefill/Decode 性能留到普通运行模块。

## 模型观察预览

官方配置显示：

```text
Qwen3-8B
Query Head = 32
KV Head = 8
→ GQA，每组 4 个 Query Head

OpenAI gpt-oss-20b
Query Head = 64
KV Head = 8
→ GQA，每组 8 个 Query Head
```

这些结论来自对应版本公开配置，完整证据与 `hidden_size` 边界放在[[06-怎样从模型配置判断Head结构|配置阅读笔记]]。

## 完成标准

学完后应能：

1. 区分 Query Head 和 KV Head；
2. 画出 MHA、GQA、MQA 的 Head 对应关系；
3. 解释共享 K/V 后为什么 Query Head 仍能产生不同 Weight；
4. 说明“共享 KV Head”不是让不同 Token 位置共用同一个数值向量；
5. 根据 `num_attention_heads` 与 `num_key_value_heads` 判断常见结构；
6. 把结构事实与 KV Cache、运行性能和质量结论分开。

下一专题：[[00-MLA与注意力变体概览|MLA 与注意力变体（扩展）]]。
