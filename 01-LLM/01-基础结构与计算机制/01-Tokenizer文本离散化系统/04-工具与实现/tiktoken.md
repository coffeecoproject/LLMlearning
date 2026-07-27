---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[工具与实现概览]]"
tags: [llm, tokenizer, openai, tiktoken]
---

# tiktoken

> [!summary]
> tiktoken 是 OpenAI 开源的快速 BPE Tokenizer 工具；日常使用主要是选择与模型匹配的 Encoding，然后进行编码、计数和解码。

## Encoding 是什么

在 tiktoken 中，可以把 **Encoding** 理解为一套已经确定的 Tokenizer 方案，其中包含可用词表、合并优先级以及特殊 Token 等信息。不同模型可能使用不同 Encoding。

```text
模型名称
→ 找到配套 Encoding
→ Encoding.encode(文本)
→ Token ID 列表
```

因此“OpenAI 都使用 tiktoken”不等于“所有 OpenAI 模型的 Token ID 都一样”。

## 【训练阶段】普通使用中没有发生什么

OpenAI 在发布某套 Encoding 之前，需要确定它的词表和 BPE 合并信息。但普通用户调用 `encode()` 时：

- 不会从当前提示词重新训练 BPE；
- 不会因为你多聊几次而修改词表；
- 不会更新 LLM 权重。

本节只讨论公开工具可确认的使用行为，不推断未公开 Encoding 的内部训练数据或完整训练流程。

## 【运行阶段】编码、计数与解码

```python
ids = encoding.encode("tiktoken is great!")
text = encoding.decode(ids)
```

官方 Cookbook 示例中，某套 Encoding 将这句话对应为若干整数，再由 `decode()` 恢复为字符串。它还展示了单个 Token 对应的字节片段，例如 `b'ikt'` 和 `b' great'`。

这揭示两个重点：

1. Token 不必等于完整单词；
2. 空格可以和后面的文字处在同一个 Token 中。

官方文档提醒：如果一个 Token 的边界并不与 UTF-8 字符边界重合，对单个 Token 使用普通 `decode()` 可能有损；观察单个 Token 时应使用 `decode_single_token_bytes()` 取得其原始字节。

## 三种 decode 必须分开

| 名称 | 所属层次 | 意义 |
|---|---|---|
| `encoding.decode(ids)` | Tokenizer | Token ID 恢复为文本 |
| Decoder-only | Transformer 架构 | 模型使用因果注意力逐 Token 预测 |
| generation decoding | 输出生成策略 | 从 logits 中贪心、采样或搜索下一个 Token |

所以 **Decoder-only 并不表示不需要 Tokenizer encode**。模型运行前仍要把用户文字编码成 Token ID；“Decoder-only”只是说 Transformer 主干没有另设一个处理源序列的 Encoder 堆栈。

## 【两阶段共同】为什么必须选对 Encoding

上下文长度、费用估算和截断都按 Token 数量工作。若 Agent 用错误 Encoding 计数，可能：

- 以为提示词放得下，实际请求却超出限制；
- 过早删除仍有价值的上下文；
- 对不同语言或代码的成本估算失真。

## 常见误解

- **“tiktoken 是一种与 BPE 并列的算法。”** 它是实现和使用 BPE Encoding 的工具。
- **“decode 说明模型使用 Encoder–Decoder。”** 两个术语处于完全不同层次。
- **“一个 Token 总能独立显示为完整字符。”** 字节级边界可能落在一个 UTF-8 字符内部。
- **“知道模型来自 OpenAI，随便选一个 Encoding 即可。”** 应使用该模型明确对应的 Encoding。

## 理解检查

1. 调用 `encode()` 时是在训练 BPE，还是使用已经确定的 Encoding？
2. 为什么单 Token 检查要关注 UTF-8 字节边界？
3. `encoding.decode()` 与 Decoder-only 有什么关系？

## 来源

- OpenAI 官方 Cookbook：[How to count tokens with tiktoken](https://developers.openai.com/cookbook/examples/how_to_count_tokens_with_tiktoken)
- 核对日期：2026-07-24。

