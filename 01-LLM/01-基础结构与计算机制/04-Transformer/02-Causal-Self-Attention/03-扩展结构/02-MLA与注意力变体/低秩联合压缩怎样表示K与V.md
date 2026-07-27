---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[MLA与注意力变体概览]]"
previous: "[[MLA为什么需要压缩KV表示]]"
next: "[[为什么MLA要解耦RoPE]]"
tags: [llm, mla, low-rank, latent, kv-compression]
---

# 低秩联合压缩怎样表示 K 与 V

> [!summary]
> MLA 让 K 和 V 共用一个较窄的、由模型学习的 KV Latent：先从 Hidden State 向下投影到潜在空间，再从潜在空间提供多头 K/V 内容；“低秩”指中间通道比原本展开表示更窄。

## 先拆开三个词

### 低秩（Low-rank）

在本节可以先理解为：

```text
不直接从宽输入一路映射到宽输出
而是强制经过一个较窄的中间空间
```

例如：

```text
8 维 → 3 维 → 12 维
```

中间只有 3 维，限制了信息变化可使用的独立方向。这里不要求先学习矩阵秩的正式定义。

### 联合（Joint）

K 和 V 不是各自保存一份互不相关的潜在表示，而是从同一个 KV Latent 出发：

```text
                 ┌→ K 内容
KV Latent ───────┤
                 └→ V 内容
```

因此它叫 Key-Value Joint Compression。

### 潜在表示（Latent Representation）

`Latent` 表示模型内部学习到、但不直接对应某个可读词义的中间数值表示。我们不能把它逐维翻译成“颜色”“人名”“语法”等固定标签。

## 一个完整的小例子

> [!example] 以下维度均为教学设定

假设某个 Token 的 Hidden State 是 8 维：

```text
h = [h1, h2, h3, h4, h5, h6, h7, h8]
```

第一步，经过学习到的向下投影：

```text
8 维 Hidden State
→ 3 维 KV Latent

c_KV = [c1, c2, c3]
```

第二步，经过学习到的向上投影，为两个 Head 提供内容表示：

```text
c_KV
├→ Head 0 的内容 Key
├→ Head 1 的内容 Key
├→ Head 0 的 Value
└→ Head 1 的 Value
```

重要的不是记住 8、3、2，而是理解因果关系：

```text
多头 K/V 不再各自直接保存全部来源信息
→ 它们共同依赖一个较窄的中间表示
```

## 这是不是先生成 K/V 再压缩

不是核心定义。MLA 的结构路径本身就是：

```text
Hidden State → KV Latent → K/V 内容
```

两个投影都属于模型参数，会一起训练。它不是在模型外部先得到完整 K/V，再用 ZIP 一类程序压缩。

## “向上投影”是不是无损解压

也不是。较窄 Latent 不可能对任意宽向量都提供通用无损还原；模型真正学习的是：

```text
怎样形成一个潜在表示
使它展开后足以完成当前模型的 Attention 任务
```

因此，“重建”或“展开”指计算路径，不承诺恢复某个预先存在的完整 K/V 原件。

## Query 为什么也可能有低秩路径

DeepSeek 的 MLA 还可以对 Query 使用单独的低秩投影：

```text
Hidden State
→ Query Latent
→ 多个 Query Head
```

它与 KV 联合压缩不是同一条 Latent：

```text
Query Latent：服务 Q 路径
KV Latent：联合服务 K/V 路径
```

KV Latent 与运行时历史信息保存直接相关；Query Latent 主要是 Query 投影的参数与激活组织，不能把 `q_lora_rank` 当成 KV Cache 宽度。

## 阶段标注

> [!info] 训练阶段
> 向下投影与向上投影的 Weight 会通过训练共同调整，使较窄通道保留对预测任务有用的信息。

> [!info] 运行阶段
> 模型使用固定投影。概念上可写成“Latent 再展开 K/V”；优化实现也可利用线性投影的结合关系，把部分展开矩阵吸收到 Query 或 Output Projection 中，避免显式生成完整 K/V。

这里字段名中的 `lora` 表示模型基础架构内置的 low-rank projection。它不是用户后来添加、用少量数据微调模型的 LoRA Adapter；两者共享“低秩”思想，但所处角色不同。

## 可选技术表达

只看结构，可以简写为：

```text
c_KV = DownKV(h)
K_content = UpK(c_KV)
V = UpV(c_KV)
```

这三行只表示函数关系，不要求现在展开矩阵乘法。

## 来源

- [DeepSeek-V2 官方论文](https://arxiv.org/abs/2405.04434)，MLA 与低秩 KV 联合压缩的原始说明。
- [DeepSeek-V3 官方推理实现](https://github.com/deepseek-ai/DeepSeek-V3/blob/main/inference/model.py)，展示 Query/KV 向下投影、归一化与向上投影。
- 核对日期：2026-07-27。

## 常见误解

- **“Low-rank 就是低精度。”** 低秩改变表示通道结构；FP8、INT8 等低精度改变数值表示方式，不是同一概念。
- **“联合压缩表示 K 与 V 完全相同。”** 它们共享 Latent，但使用不同的向上投影，输出角色仍不同。
- **“Latent 的每一维都有固定人类语义。”** 其含义由分布式表示和训练共同形成。
- **“向上投影能无损恢复任意输入。”** 它学习任务所需的展开，不是通用无损解压。
- **“Query Latent 和 KV Latent 是同一个东西。”** 两者服务不同路径，配置字段也不同。
- **“配置里的 `lora_rank` 表示额外安装了 LoRA 微调 Adapter。”** 在 DeepSeek MLA 中，它是基础模型结构字段。

## 理解检查

1. “低秩”在本节最直观地表示什么？
2. “联合”为什么不表示 K 和 V 数值完全相同？
3. MLA 为什么不是先生成完整 K/V 再调用外部压缩程序？
4. Query Latent 与 KV Latent 的职责有什么区别？

下一篇：[[为什么MLA要解耦RoPE]]。
