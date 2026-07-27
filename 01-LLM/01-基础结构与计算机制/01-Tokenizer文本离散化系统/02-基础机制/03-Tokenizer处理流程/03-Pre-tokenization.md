---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Tokenizer处理流程概览|Tokenizer处理流程概览]]"
previous: "[[02-Normalization|Normalization]]"
next: "[[04-Tokenization-Model|Tokenization-Model]]"
tags: [llm, tokenizer, pre-tokenization]
---

# Pre-tokenization

## 一句话理解

> Pre-tokenization 在核心算法前按照空格、标点、数字或正则规则做初步分段，为后续切分限定处理边界，但它产生的片段通常还不是最终 Token。

## 示例

```text
Hello, world! 2026
```

某个实现可能初步分成：

```text
Hello | , |  world | ! |  2026
```

注意前导空格可能被保留，因为 `apple` 与 ` apple` 在一些词表中对应不同片段。

## 为什么需要预切分？

如果允许核心合并算法跨任意边界工作，它可能把不希望组合的跨词或跨类别片段合并。Pre-tokenization 可以定义哪些区域分别交给后续算法。

## 中文和代码

中文通常没有空格词界，因此仅按空格预切分帮助有限。代码则可能需要处理下划线、括号、缩进和运算符。不同语料目标会导致不同规则。

## 它为什么不是最终切分？

`Hello` 这个初步片段仍可能被核心模型切成：

```text
Hell | o
```

或者整体成为一个 Token。最终结果由 Vocabulary 和 Tokenization Model 决定。

## 常见误解

- Pre-tokenization 不等于传统语言学分词。
- 不是所有 Tokenizer 都有明显独立的预切分阶段。
- 按空格切分不能解决所有语言和代码问题。

## 理解检查

1. Pre-tokenization 和最终 Tokenization 有什么区别？
2. 为什么某些 Token 会把前导空格包含在内部？
3. 中文为什么不能只依靠空格预切分？

下一篇：[[04-Tokenization-Model|Tokenization Model]]。
