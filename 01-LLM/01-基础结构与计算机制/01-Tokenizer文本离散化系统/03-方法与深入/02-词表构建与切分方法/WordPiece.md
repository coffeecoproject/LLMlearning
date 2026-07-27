---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[词表构建与切分方法概览]]"
previous: "[[BPE]]"
next: "[[Unigram]]"
tags: [llm, tokenizer, wordpiece, vocabulary]
---

# WordPiece

> [!summary]
> WordPiece 也是一种子词方法：训练时建立有限的子词词表，运行时从当前位置开始，反复寻找词表中能匹配的最长片段。

## 先看运行结果

以 BERT 风格的 WordPiece 为例，一个英文词可能被表示为：

```text
playing
→ [play] [##ing]
```

`##` 通常表示该片段出现在单词内部、需要接在前一个片段后面。它不是原文真的包含两个井号，而是 Tokenizer 用来区分位置的记号。

> [!note]
> 上例用于说明形式，真实切分必须由具体词表确定。

## 【Tokenizer 训练阶段】从小单位建立词表

WordPiece 与 BPE 的总体方向相似：

```text
训练语料
→ 把词拆成初始小单位
→ 评估哪些相邻片段适合组成新片段
→ 逐步扩大词表
→ 达到目标词表规模
```

BPE 的经典入门版本倾向于合并出现频率最高的相邻对；WordPiece 的公开描述通常强调候选片段对整体语言建模价值的改善，而不是只看相邻频率。

### 已知与不确定边界

Google 没有开源原始 WordPiece 训练代码。Hugging Face 官方课程明确把其训练过程描述为根据论文信息做出的合理复现，并提醒可能无法百分之百还原原始实现。

因此应掌握核心差别，但不要记住某一条教学评分公式后，就认定所有 WordPiece 训练器必须完全一样。

## 【Tokenizer 使用阶段】最长匹配怎样工作

假设示意词表包含：

```text
play
p
##lay
##ing
##i
##n
##g
```

对 `playing` 编码：

```text
从单词开头寻找最长可用片段 → play
剩余部分变成词内形式       → ##ing
最终结果                   → [play] [##ing]
```

如果没有 `play`，但存在 `p` 和 `##lay`，结果可能变成：

```text
[p] [##lay] [##ing]
```

这是一种从左向右的最长匹配策略。它使用训练完成后的词表，不会在处理这句话时重新学习子词。

## 无法完整切分时

经典 BERT 风格 WordPiece 中，如果一个词的某个剩余部分找不到任何有效片段，整个词可能变成：

```text
[UNK]
```

这与具有字节回退能力的现代 Tokenizer 不同。后者通常能继续拆到字节，从而避免完全未知。

## WordPiece 与 Subword 的关系

```text
Subword：一种文本表示粒度
WordPiece：学习并选择 Subword Token 的方法
```

WordPiece 最终产生的 `play`、`##ing` 仍然都是 Token。

## 常见误解

- **“`##` 是用户原文的一部分。”** 它通常是词内位置标记。
- **“WordPiece 就是最长单词切分。”** 它只在已训练词表中寻找最长可用子词。
- **“WordPiece 和 BPE 运行方式完全相同。”** BPE 常按已学合并优先级执行；WordPiece 常用词表最长匹配。
- **“WordPiece 能切出语言学上正确的词根。”** 它学习的是统计上有用的片段，不保证符合语言学分析。

## 理解检查

1. `##ing` 中的 `##` 表示什么？
2. WordPiece 处理新文本时是否会重新训练词表？
3. WordPiece 与 Subword 分别属于方法还是表示粒度？

## 来源

- Hugging Face LLM Course：[WordPiece tokenization](https://huggingface.co/learn/llm-course/chapter6/6)
- 核对日期：2026-07-24。

下一篇：[[Unigram|Unigram]]。
