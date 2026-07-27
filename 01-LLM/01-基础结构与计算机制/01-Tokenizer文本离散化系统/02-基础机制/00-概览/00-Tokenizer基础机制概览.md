---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Tokenizer文本离散化系统概览|Tokenizer文本离散化系统概览]]"
previous: "[[00-Tokenizer框架速览概览|Tokenizer框架速览概览]]"
next: "[[00-Token概览|Token概览]]"
tags: [llm, tokenizer, token, vocabulary, encoding]
---

# Tokenizer 基础机制

> [!summary]
> 这一层解释 Tokenizer 实际处理哪些对象、依赖哪些资源，以及一次编码和解码怎样完整发生。

## 阅读顺序

1. [[00-Token概览|Token]]：模型处理的离散对象是什么；
2. [[00-Vocabulary与Token ID概览|Vocabulary 与 Token ID]]：有限词表怎样给 Token 分配索引；
3. [[00-Tokenizer处理流程概览|Tokenizer 处理流程]]：文本怎样经过预处理、切分、查表和输入整理。

```text
原始文本
→ 处理与切分
→ Token
→ Vocabulary Lookup
→ Token ID
```

## 这一层需要分清

- Token 是离散对象；
- Vocabulary 是 Tokenizer 使用的有限资源；
- Token ID 是 Token 在这套词表中的整数索引；
- Encode 把文本变成 ID，Decode 尝试把 ID 恢复为文本；
- Embedding 在 Tokenizer 之后，不属于 Tokenizer 内部。

## 暂时可以跳过

BPE、WordPiece、Unigram 的规则学习过程，以及 tiktoken、SentencePiece、Hugging Face Tokenizers 的软件实现，都不是理解基本输入输出链的前置条件。

下一篇：[[00-Token概览|Token]]。
