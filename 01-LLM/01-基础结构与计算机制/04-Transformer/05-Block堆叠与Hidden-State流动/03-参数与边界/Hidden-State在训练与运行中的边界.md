---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[Block参数与边界概览]]"
previous: "[[num-hidden-layers与每层参数]]"
next: "[[怎样理解不同层的功能]]"
tags: [llm, hidden-state, activations, training, inference]
---

# Hidden State 在训练与运行中的边界

> [!summary]
> Hidden States 在训练和运行的前向计算中都会逐层产生；训练需要它们参与梯度计算，普通运行只把它们作为当前请求的临时状态，两者都不意味着自动永久修改模型参数。

## 两阶段共同部分

```text
当前输入
→ Embedding
→ Block 1 Hidden States
→ Block 2 Hidden States
→ ……
→ Final Hidden States
```

同一模型面对不同输入会产生不同 Hidden States。

## LLM 训练阶段

训练时：

```text
前向计算 Hidden States
→ 得到 Logits 与 Loss
→ 反向传播计算梯度
→ 优化器更新模型参数
```

为了反向传播，系统通常需要保留部分中间激活，或之后重新计算它们。具体的 Activation Checkpointing、显存优化和分布式训练属于训练工程。

真正跨训练步骤保存的是更新后的模型参数与优化器状态等，不是把每个样本的全部 Hidden States 当成知识库永久保存。

## LLM 运行阶段

普通运行时：

```text
模型参数固定
→ 根据当前 Prompt 计算 Hidden States
→ 用于产生本次输出
→ 请求生命周期结束后通常释放相应临时状态
```

上下文看起来像“模型记得刚才说过的话”，通常是因为对话历史再次作为输入上下文发送，或外部系统保存了记忆；不是普通前向计算自动改写了模型权重。

## Hidden State 与 KV Cache

```text
Hidden State
→ 某层、某位置当前的完整内部表示

KV Cache
→ 为后续生成缓存各 Attention 层已经计算的 Key 和 Value
```

KV Cache 来源于逐层计算，但不是“保存全部 Hidden States 的长期记忆库”。它服务当前生成过程的计算复用，通常由推理服务器管理。

## Hidden State 与参数

| 对象 | 是否随输入改变 | 普通运行是否更新 | 主要生命周期 |
|---|---:|---:|---|
| 模型参数 | 通常否 | 否 | 模型权重长期保存 |
| Hidden States | 是 | 每次重新计算 | 当前前向过程 |
| KV Cache | 是 | 随生成追加/管理 | 当前生成请求 |
| 外部 Agent Memory | 由系统决定 | 可写入外部存储 | 模型外部系统 |

## 常见误解

- Hidden State 变化不等于模型参数变化。
- 训练时产生中间激活不等于逐样本永久保存知识。
- KV Cache 不是模型的长期记忆参数。
- 多轮对话记忆可能来自重新发送历史或外部存储。

## 理解检查

1. 为什么同一模型参数能对不同 Prompt 产生不同 Hidden States？
2. 训练为什么需要中间激活，而运行通常不执行参数更新？
3. Hidden State 与 KV Cache 的职责有什么不同？

下一篇：[[怎样理解不同层的功能|怎样理解不同层的功能]]。
