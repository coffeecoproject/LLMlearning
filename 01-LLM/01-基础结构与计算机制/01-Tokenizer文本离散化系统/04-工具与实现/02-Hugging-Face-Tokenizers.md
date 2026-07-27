---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-工具与实现概览|工具与实现概览]]"
tags: [llm, tokenizer, hugging-face]
---

# Hugging Face Tokenizers

> [!summary]
> Hugging Face 提供的是一套可组合的 Tokenizer 实现生态；`AutoTokenizer` 是按模型加载正确实现与文件的入口，不是一种新的切分算法。

## 两个经常混用的名字

1. **`tokenizers` 库**：Rust 实现的高性能库，可构造和训练 Tokenizer 流水线。
2. **Transformers 中的 `AutoTokenizer`**：根据模型仓库的配置，选择并加载配套 Tokenizer。

所以：

```text
AutoTokenizer
≠ BPE / WordPiece / Unigram
≠ 一个所有模型共享的统一词表
```

## 【训练阶段】可以组合并训练完整流水线

Hugging Face Tokenizers 将流程拆成可配置部件：

```text
Normalizer
→ Pre-tokenizer
→ Model（BPE / WordPiece / Unigram 等）
→ Trainer
→ Post-processor
→ Decoder
```

训练器读取语料，学习所选模型需要的词表或规则。之后再保存结果。这里的 `Model` 指 Tokenizer 内部的切分模型，不是完整 LLM。

## 【运行阶段】加载后准备模型输入

实际使用开源模型时更常见的是：

```python
tokenizer = AutoTokenizer.from_pretrained("某个模型仓库")
inputs = tokenizer("我喜欢苹果")
```

运行结果通常不只有 `input_ids`，还可能包括 `attention_mask` 等模型输入。具体字段取决于任务和模型配置。

官方 Transformers 文档还区分：

- Python 实现的 Tokenizer；
- 基于 Rust `tokenizers` 库的 Fast Tokenizer。

Fast 版本除批量处理更快外，通常还能记录原始字符位置与 Token 位置之间的对应关系。

## 【两阶段共同】`AutoTokenizer` 为什么重要

模型仓库中可能同时存在词表、合并规则、特殊 Token 和处理配置。`AutoTokenizer` 的价值不是替你发明算法，而是尽量按照仓库记录把正确组合还原出来。

```text
模型名称
→ 读取配置
→ 选择 Tokenizer 类
→ 加载具体文件
→ 得到模型配套实例
```

## 例子：相同调用，不同结果

下面两行代码形式相同：

```python
a = AutoTokenizer.from_pretrained("模型 A")
b = AutoTokenizer.from_pretrained("模型 B")
```

但 A、B 可能使用不同算法、词表、特殊 Token 和聊天模板，因此对“我喜欢苹果🍎”输出不同 ID 和长度。这说明真正决定结果的是“库 + 模型资产”，不只是 `AutoTokenizer` 这个函数名。

## 常见误解

- **“Hugging Face Tokenizer 就是一种算法。”** 它是工具生态，可承载多种算法。
- **“Fast 和 Slow 一定切得不同。”** 它们的目标通常是实现同一个 Tokenizer；速度和附加能力不同不等于规则必然不同。
- **“`AutoTokenizer` 会自动训练一个新词表。”** `from_pretrained` 的主要作用是加载已有资产。
- **“新增 Token 只改词表即可。”** 若要让 LLM 使用新增 ID，通常还需同步调整模型 Embedding 等参数并进行训练。

## 理解检查

1. `AutoTokenizer` 为什么不是 WordPiece 的竞争算法？
2. `tokenizers` 库中的 Trainer 在哪个阶段工作？
3. 为什么相同的 `AutoTokenizer` 调用形式可能产生不同 Token？

## 来源

- Hugging Face Transformers 官方文档：[Tokenizer](https://huggingface.co/docs/transformers/main_classes/tokenizer)
- 核对日期：2026-07-24。

