---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Prefill与首次预测概览|Prefill与首次预测概览]]"
previous: "[[00-Prefill与首次预测概览|Prefill与首次预测概览]]"
next: "[[02-为什么读取最后有效位置的Logits|为什么读取最后有效位置的Logits]]"
tags: [llm, inference, prefill, forward-pass]
---

# Prefill 是什么

> [!summary]
> Prefill 是“先读完当前可见上下文”的阶段：模型对输入 Token 进行一次完整前向计算，为第一个输出 Token 做准备。

## 为什么必须先有 Prefill

用户发送的 Prompt 可能已经包含数十、数千甚至更多 Token。模型若要预测回答的第一个 Token，必须先让这些输入经过：

```text
Embedding 与 Position
→ 多层 Transformer Block
→ Final Norm 与 LM Head
```

同时，每层 Attention 会为这些已处理位置形成后续可复用的 K、V 状态。

## 一个简化例子

输入 Token：

```text
[今天] [天气] [很好]
```

Prefill 会处理三个输入位置。由于 Causal Attention：

```text
[今天]     只能读取 [今天]
[天气]     可以读取 [今天][天气]
[很好]     可以读取 [今天][天气][很好]
```

虽然各位置遵守“不能看未来”，整段已知输入的矩阵计算仍可高度并行。因果限制描述的是信息可见性，不等于程序必须把 Prompt 一个 Token 一个 Token 慢慢算完。

## Prefill 的两个主要结果

### 1. 第一次预测需要的 Logits

最后有效输入位置的 Final Hidden State 经过 Output Layer，给出下一个 Token 的候选分数。

### 2. 后续 Decode 需要的运行状态

各层 Attention 的历史 K、V 会进入 KV Cache。后续新增 Token 可以读取这些缓存，不必每一步重新为全部历史 Token 计算 K、V。

## Prefill 不是什么

- 不是训练，不计算参数更新；
- 不是把 Prompt 写进长期记忆；
- 不是一次生成完整答案；
- 不是 Chat Template 或 Tokenizer；这两者在它之前准备输入。

## 输入长度为什么影响首 Token 等待时间

Prefill 需要处理现有输入。一般来说，输入越长，需要完成的 Attention 与 FFN 计算越多，因此用户等待第一个输出 Token 的时间通常会增加。具体速度还受硬件、模型结构和 Runtime 优化影响。

## 理解检查

1. Prefill 为什么发生在第一个输出 Token 之前？
2. Prompt 内各位置不能看未来，为什么计算仍可并行？
3. Prefill 通常给后续阶段留下哪两类结果？

下一篇：[[02-为什么读取最后有效位置的Logits|为什么读取最后有效位置的 Logits]]。
