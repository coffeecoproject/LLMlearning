---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Tokenizer文本离散化系统概览]]"
previous: "[[Tokenizer处理流程概览]]"
next: "[[文本表示路线概览]]"
tags: [llm, tokenizer, subword, bpe, wordpiece, unigram, optional]
---

# Tokenizer 方法与深入

> [!summary]
> 这一层解释 Tokenizer 可以选择什么文本表示粒度，以及 BPE、WordPiece、Unigram 怎样产生并使用自己的词表与切分规则。

## 两类问题不要混在一起

```text
表示路线
→ 倾向使用单词、字符、子词还是字节作为基础表示

构建与切分方法
→ 怎样从语料形成词表，并在使用时决定切分结果
```

## 阅读顺序

1. [[文本表示路线概览|文本表示路线]]：Word、Character、Subword、Byte；
2. [[词表构建与切分方法概览|词表构建与切分方法]]：BPE、WordPiece、Unigram；
3. [[工具与实现概览|工具与实现]]：按需观察软件和模型文件；
4. [[影响与边界概览|影响与复习]]：Token 数量、语言差异、未知输入和能力边界。

这里不展开 LLM 参数训练。Tokenizer 的规则或词表如何构建，与 LLM 怎样通过 Loss 更新权重，是两套不同训练过程。

下一篇：[[文本表示路线概览|文本表示路线]]。
