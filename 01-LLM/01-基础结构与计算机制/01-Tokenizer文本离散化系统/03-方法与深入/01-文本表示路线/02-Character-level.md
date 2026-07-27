---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-文本表示路线概览|文本表示路线概览]]"
previous: "[[01-Word-level|Word-level]]"
next: "[[03-Subword-level|Subword-level]]"
tags: [llm, tokenizer, character-level, unicode]
---

# Character-level：按字符表示

## 一句话理解

> Character-level 用较小的字符单位拼出文本，降低完整未知词问题，但会产生更长序列，而且计算机中的“一个字符”并不总等于屏幕上的一个符号。

## 从完整单词退回字符

如果词表不需要保存 `unhappiness`，只要保存组成它的字符，就能表示这个新词：

```text
u | n | h | a | p | p | i | n | e | s | s
```

这解决了 Word-level 的一部分 OOV 问题：一个没见过的单词，仍可以由已知字符组成。

## 为什么序列会变长？

Word-level 可能把 `unhappiness` 当作 1 个 Token，字符级需要 11 个。模型处理的序列位置因此增加。

更长的序列意味着：

- 同样的上下文窗口能容纳的原文更少；
- 模型需要跨更多位置组合出词和短语；
- Attention 等计算需要处理更多位置关系。

因此字符级用更小词表换来了更长序列。

## 中文中的直觉优势

示意：

```text
我喜欢苹果
→ 我 | 喜 | 欢 | 苹 | 果
```

汉字本身经常携带一定信息，所以中文字符级看起来比英文字母级更自然。但“喜欢”和“苹果”的组合意义仍需要模型跨位置形成。

## “字符”其实有多种边界

日常说的字符，可能指不同对象：

- Unicode Code Point：Unicode 分配的抽象编号；
- Grapheme Cluster：用户视觉上认为的一个完整符号；
- 编程语言字符串中的一个存储单元。

它们不总是一一对应。例如某些带音调字母可以是一个预组合 Code Point，也可以由基础字母加组合符号构成。某些家庭 Emoji 由多个 Code Point 通过连接符组合，但屏幕上看起来像一个图形。

所以“一个屏幕字符等于一个 Token”并不是稳定规则。

## 规范化为什么会相关？

两个视觉相同的字符串，底层 Unicode 表示可能不同。Tokenizer 可能先做 Normalization，把某些等价形式统一；也可能保留差异。

这意味着字符级路线仍需要回答：

```text
按哪个 Unicode 边界切？
是否先规范化？
哪些字符进入有限词表？
词表外字符怎样处理？
```

## 字符级是否彻底消灭未知输入？

不一定。如果词表只收录有限字符，仍可能遇到没收录的文字或符号。Unicode 字符集合虽然有限定义，但规模很大且持续演进。工程上常用字节级基础或 byte fallback 提供更稳健的兜底。

## 优势与代价

优势：

- 词表通常比完整单词词表小；
- 能组合出大量训练时未见过的单词；
- 对拼写变化保留的信息比统一 `[UNK]` 更多。

代价：

- 序列显著变长；
- 一个词的意义需要跨多个位置组合；
- Unicode 和视觉字符边界并不简单；
- 有限字符词表仍可能存在覆盖缺口。

## 常见误解

- 汉字、Unicode Code Point、UTF-8 字节和 Token 是四个不同层面的对象。
- Character-level 不保证词表只有几百项；多语言字符集合可能很大。
- “能拼出来”只表示可编码，不表示模型一定理解该词。

## 理解检查

1. Character-level 怎样缓解完整未知词问题？
2. 为什么它会消耗更多上下文位置？
3. 一个看起来完整的 Emoji 为什么不一定只有一个 Unicode Code Point？

下一篇：[[03-Subword-level|Subword-level：按子词表示]]。
