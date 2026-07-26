---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[Token概览]]"
previous: "[[Token到底是什么]]"
next: "[[Vocabulary与Token ID概览]]"
tags: [llm, token, subword, byte, special-token]
---

# Token 可能对应哪些文本单位

> [!summary]
> Token 没有固定的自然语言粒度：一个 Token 可能对应完整词、子词、字符、字节片段、空白或特殊控制符号。

## 常见形态

| 形态 | 教学示例 | 需要注意 |
|---|---|---|
| 完整词或常见片段 | `apple`、`模型` | 不保证在所有 Tokenizer 中都是一个 Token |
| 子词 | `un`、`ing` | 不一定是真正的语言学词根或词缀 |
| 单个或多个字符 | `猫`、`人工智能` | 字符数量与 Token 数不固定对应 |
| 空白相关片段 | ` hello`、换行、缩进 | 空格可能独立，也可能与文字合并 |
| 数字片段 | `20`、`2026` | 一个数可能被拆成多个 Token |
| 字节片段 | UTF-8 字节的一部分 | 单个 Token 未必能独立显示为正常字符 |
| 特殊 Token | `<bos>`、`<eos>` | 主要表达协议结构，而非普通文本 |

这些例子只说明可能性，不代表任何真实模型的具体切分。

## Token 与其他单位不要混为一谈

```text
Unicode Code Point → 字符标准中的编号
Grapheme Cluster   → 人眼通常看作一个书写字符的组合
Byte               → 文本编码后的存储单位
Word / Morpheme    → 语言学单位
Token              → 具体 Tokenizer 词表中的符号
```

它们有时重合，但没有固定的一一对应关系。一个 Emoji 可能由多个 Unicode Code Point 和多个 UTF-8 字节组成，也可能被表示成一个或多个 Token。

## 什么决定 Token 边界

Token 边界通常共同取决于：

- 基础表示路线，如字符或字节；
- 训练语料中片段的统计规律；
- 目标词表大小；
- BPE、WordPiece 或 Unigram 等方法；
- Normalization 和 Pre-tokenization 规则；
- 空格、数字以及特殊 Token 配置。

所以：

```text
相同文本 + 不同 Tokenizer → 可能得到不同 Token 序列
相同文本 + 同一版本与配置 → 通常得到确定的结果
```

## 教学例子

文本：

```text
 learning😊
```

某个假想 Tokenizer 可能切成：

```text
[" learn", "ing", "😊"]
```

另一个可能切成：

```text
[" ", "l", "e", "a", "r", "n", "i", "n", "g", <若干字节片段>]
```

这不是谁绝对正确，而是两套词表和表示路线不同。

## 开放模型观察：Llama 3

Meta Llama 3 官方 Tokenizer 实现基于 TikToken，并定义了包括 `<|begin_of_text|>`、`<|end_of_text|>` 和 `<|eot_id|>` 在内的特殊 Token。这说明真实词表既可以包含普通文本片段，也可以包含对话协议符号。

来源：[Meta Llama 3 官方 `tokenizer.py`](https://github.com/meta-llama/llama3/blob/main/llama/tokenizer.py)，核对日期：2026-07-24。这里只观察 Llama 3，不推断其他版本完全相同。

## 常见误解

- **“中文一个字就是一个 Token。”** 不一定。
- **“英文一个单词就是一个 Token。”** 不一定。
- **“一个 Emoji 一定是一个 Token。”** 不一定。
- **“Token 边界就是语义边界。”** Token 边界首先是具体 Tokenizer 的工程与统计结果。

## 理解检查

1. 为什么字符数不能直接换算成 Token 数？
2. 空格为什么可能影响切分？
3. 特殊 Token 与普通文本 Token 的共同点和区别是什么？

下一节：[[Vocabulary与Token ID概览|Vocabulary 与 Token ID]]。
