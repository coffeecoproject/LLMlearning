---
type: subtopic-index
module: 1
status: complete
audience: non-specialist
parent: "[[Tokenizer文本离散化系统概览]]"
previous: "[[Tokenizer文本离散化系统概览]]"
next: "[[Tokenizer基础机制概览]]"
tags: [llm, tokenizer, framework, beginner]
---

# Tokenizer 一页看懂

> [!summary]
> Tokenizer 是模型外部的文本转换系统：它把开放文本切成有限词表能够表示的 Token，再把 Token 映射成 Token ID。

## 它位于哪里

```text
用户文字
→ Tokenizer 编码
→ Token ID 序列
→ Embedding
→ LLM
```

模型生成结束时还有反方向：

```text
模型选出的 Token ID
→ Tokenizer 解码
→ 用户看到的文字
```

## 为什么需要它

神经网络不能直接把任意字符串送进矩阵计算。它需要有限、稳定的离散编号，而现实文本包含不同语言、标点、数字、代码和 Emoji。

Tokenizer 在二者之间建立转换规则：

```text
开放文本
→ 有限 Token 集合
→ 稳定整数编号
```

## 一个教学例子

假设某套 Tokenizer 把：

```text
我喜欢苹果
```

处理为：

```text
[我] [喜欢] [苹果]
→ [41, 208, 930]
```

另一套 Tokenizer 可能切成不同片段。这里的 Token 和 ID 都是人为示意，不代表任何真实模型。

## 完整 Tokenizer 最少包含什么

```text
切分与处理规则
+ Vocabulary（有限词表）
+ Token 到 ID 的映射
+ 特殊 Token 配置
+ 编码与解码规则
```

因此 Tokenizer 不只是“按空格切词”，也不只是一个 Vocabulary 文件。

## 两个阶段不要混淆

```text
Tokenizer 构建阶段
→ 从语料与配置产生词表、切分规则等资产

Tokenizer 使用阶段
→ 使用已经固定的资产编码或解码文本
```

LLM 训练和普通运行都会使用 Tokenizer，但通常不会在每次输入时重新构建它。

## 它没有做什么

- Token ID 只是词表索引，不是语义大小；
- Tokenizer 不把 ID 变成向量，那是 Embedding 的工作；
- Tokenizer 不理解完整上下文，那是后续 Transformer 的工作；
- BPE、WordPiece、Unigram 是构建和切分方法，不是三种 Token 类型；
- Tokenizer 的 `decode()` 与 Decoder-only 架构没有直接关系。

## 框架层检查

1. Tokenizer 的直接输入和输出分别是什么？
2. Token 与 Token ID 有什么区别？
3. 为什么不同模型必须使用与自己配套的 Tokenizer？
4. Tokenizer 构建与使用为什么不是同一件事？

能回答这四题，就可以先进入 [[Embedding框架速览概览|Embedding 一页看懂]]。需要理解内部机制时，再读 [[Tokenizer基础机制概览|Tokenizer 基础机制]]。
