---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[FFN边界与复习概览]]"
previous: "[[FFN边界与复习概览]]"
next: "[[Block堆叠与Hidden-State流动概览]]"
tags: [llm, ffn, parametric-knowledge, memory]
---

# FFN 与模型知识的关系

> [!summary]
> FFN 不会在运行时访问外部世界，但其参数在训练中吸收了大量文本模式；当前上下文化表示经过 FFN 时，会受到这些已学习参数模式的影响。

## 修正常见比喻

常见说法：

```text
Attention 负责 Token 与 Token 的关系
FFN 负责 Token 与外部世界的关系
```

更准确的版本：

```text
Attention
→ 组织当前输入中不同 Token 位置的关系

FFN
→ 使用训练形成的参数化模式，加工当前位置的上下文化表示
```

“外部世界”如果指其他 Token，属于 Attention；如果指训练数据描述过的事实和规律，它们已经通过训练影响模型参数，不是运行时仍在模型外部等待查询。

## 三种信息来源

| 信息 | 来源与进入方式 |
|---|---|
| 当前上下文 | Prompt、系统指令、工具结果，经 Tokenizer 后进入模型 |
| 参数化知识 | 训练数据长期塑造 FFN、Attention、Embedding 等权重 |
| 外部实时信息 | 搜索、数据库、文件或 API 取得后，再加入当前上下文 |

```text
FFN 使用参数化模式
≠ FFN 正在查询实时外部信息
```

## “法国的首都是”示例

```text
Attention
→ 把“法国”“首都”等当前线索汇集到预测位置

FFN
→ 当前表示经过训练形成的参数模式
→ 产生有助于事实关联调用的特征变化

更多 Block + Output Layer
→ 最终可能提高“巴黎”的 Logit
```

不能简化成：

```text
FFN数据库[法国首都] → 巴黎
```

答案依赖多层 Attention、FFN、Residual 和 Output Layer 的共同作用。

## Key-Value Memory 研究视角

研究者曾把 Transformer FFN 分析成未归一化的 Key-Value Memory：某些中间方向像是在检测输入模式，输出方向像是在向 Residual Stream 写入相关变化。

这里的 Key/Value：

- 不是 Attention 为每个 Token 计算的 K/V；
- 不是运行时 KV Cache；
- 不是所有现代 SwiGLU、MoE 或闭源模型都公开承诺的精确数据结构。

> [!source]
> Geva 等：《Transformer Feed-Forward Layers Are Key-Value Memories》，[arXiv:2012.14913](https://arxiv.org/abs/2012.14913)。Meng 等：《Locating and Editing Factual Associations in GPT》，[arXiv:2202.05262](https://arxiv.org/abs/2202.05262)。核对日期：2026-07-27。

## 为什么不能把 FFN 直接叫知识库

- 模型表示分布在多个层和组件；
- 事实关联可能重叠且没有唯一地址；
- 模型可能记错、混淆或幻觉；
- 运行一次不会永久写入参数；
- 知识库通常强调稳定记录和精确查询，神经参数不具备相同保证。

## 最稳妥的表述

> Attention 主要把当前上下文信息送到合适位置；FFN 使用训练形成的参数化变换，对每个位置的上下文化表示进一步加工。FFN 是事实关联和参数化模式的重要参与者，但不是独立、精确、实时更新的外部知识库。

## 理解检查

1. “外部”指其他 Token 和指现实知识时，分别应怎样理解？
2. 参数化知识与外部实时信息有什么不同？
3. FFN 的 Key-Value Memory 视角为什么不是 Attention K/V？
4. 为什么 FFN 与事实关联有关，却不能称为精确数据库？

FFN 专题结束。下一节：[[Block堆叠与Hidden-State流动概览|Block 堆叠与 Hidden State 流动]]。
