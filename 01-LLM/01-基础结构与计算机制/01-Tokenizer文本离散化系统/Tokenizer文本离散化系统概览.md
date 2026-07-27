---
type: topic-index
module: 1
status: complete
audience: non-specialist
parent: "[[01-LLM/01-基础结构与计算机制/基础结构与计算机制大纲]]"
tags: [llm, token, tokenizer]
---

# Tokenizer 文本离散化系统

> [!summary]
> Tokenizer 位于原始文本与模型数值输入之间，负责把文本转换成 Token ID，并把生成出的 Token ID 解码回文字。

## 按学习目标选择入口

### 只看框架

阅读：[[Tokenizer框架速览概览|Tokenizer 一页看懂]]。

读完只需掌握：

```text
文字 → Token → Token ID
```

以及 Tokenizer 构建与 Tokenizer 使用不是同一阶段。

### 理解基础机制

进入：[[Tokenizer基础机制概览|Tokenizer 基础机制]]。

这里解释 Token、Vocabulary、Token ID 和完整编码/解码流程。

### 继续深入

进入：[[Tokenizer方法与深入概览|Tokenizer 方法与深入]]。

这里讨论文本表示路线、BPE、WordPiece、Unigram、工具实现及影响边界。

## 系统结构

```text
Tokenizer
├── 处理对象：Token
├── 使用资源：Vocabulary 与切分规则
├── 输出表示：Token ID
├── 表示路线：Word / Character / Subword / Byte
├── 构建与切分方法：BPE / WordPiece / Unigram
├── 编码与解码流程
├── 软件与模型文件
└── 对长度、效率和文本覆盖的影响
```

## 阶段边界

```text
Tokenizer 构建阶段
→ 产生 Vocabulary、规则、分数和特殊 Token 配置

LLM 训练阶段
→ 使用固定 Tokenizer 编码训练文本

LLM 运行阶段
→ 使用配套 Tokenizer 编码输入、解码输出
```

Tokenizer 构建不是 LLM 参数训练；Tokenizer 使用也不是 Embedding 或 Transformer 计算。

## 当前内容

- [x] [[Tokenizer框架速览概览|框架速览]]
- [x] [[Tokenizer基础机制概览|基础机制]]
- [x] [[Tokenizer方法与深入概览|方法与深入]]
- [x] [[工具与实现概览|工具与实现]]
- [x] [[影响与边界概览|影响与复习]]

框架路线下一站：[[Embedding框架速览概览|Embedding 一页看懂]]。
