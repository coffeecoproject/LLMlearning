---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Tokenizer基础机制概览|Tokenizer基础机制概览]]"
previous: "[[00-词表构建与切分方法概览|词表构建与切分方法概览]]"
next: "[[01-编码与解码全景|编码与解码全景]]"
tags: [llm, tokenizer, encoding, decoding]
---

# Tokenizer 处理流程概览

## 一句话理解

> 完整 Tokenizer 是一条文本与 Token ID 之间的双向流水线；BPE 只是其中负责切分的一种核心方法，不等于整个 Tokenizer。

## 完整数据流

```text
聊天消息
→ Chat Template（上游协议）
→ 原始输入文本
→ Normalization
→ Pre-tokenization
→ 基础单位转换
→ Tokenization Model
→ Post-processing / Special Tokens
→ Vocabulary Lookup
→ Token IDs
→ Tokenizer 核心编码在 Token IDs 处完成
→ Padding / Truncation / Attention Mask（相邻的输入整理接口）
→ LLM 接口
```

任意一组需要恢复成文本的 Token ID 都走反向流程：

```text
待解码的 Token IDs
→ 查 Vocabulary
→ Token 或字节片段
→ 拼接与解码
→ 用户看到的文本
```

## 四条边界

1. Chat Template 在 Tokenizer 上游，负责把角色化消息组织成模型协议。
2. BPE、WordPiece、Unigram 是 Tokenization Model，不代表完整 Tokenizer。
3. Padding、Truncation 和 Attention Mask 常由 Tokenizer 库一并提供，但属于模型输入整理，不是核心切分算法。
4. 不同实现可能在 Token 层或 ID 层加入特殊 Token，因此流程图表达职责关系，不要求所有软件逐行采用相同顺序。

## 子结构

1. [[01-编码与解码全景|编码与解码全景]]
2. [[02-Normalization|Normalization]]
3. [[03-Pre-tokenization|Pre-tokenization]]
4. [[04-Tokenization-Model|Tokenization Model]]
5. [[05-Special-Token与Post-processing|Special Token 与 Post-processing]]
6. [[06-Vocabulary-Lookup与输入整理边界|Vocabulary Lookup 与输入整理边界]]
7. [[07-Decode与文本恢复|Decode 与文本恢复]]
8. [[08-可逆性与边界|可逆性与边界]]

## 通用框架不等于统一实现

不同 Tokenizer 可能省略或合并某些阶段，也可能采用不同顺序和命名。因此这套结构用于分析系统，不应被误解为所有库都必须拥有八个同名函数。

## 理解检查

1. 为什么 BPE 不能代表完整 Tokenizer？
2. Chat Template 为什么放在 Tokenizer 上游？
3. Padding 为什么不是 BPE 的一部分？
4. 编码与解码是否一定完全可逆？
