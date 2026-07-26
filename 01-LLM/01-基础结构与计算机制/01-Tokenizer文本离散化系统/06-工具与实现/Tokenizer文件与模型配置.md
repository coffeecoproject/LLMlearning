---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[工具与实现概览]]"
tags: [llm, tokenizer, configuration]
---

# Tokenizer 文件与模型配置

> [!summary]
> 模型目录里的 Tokenizer 文件是训练阶段确定、运行阶段加载的契约；不同仓库的文件组合不完全相同，不能只凭文件名猜全部行为。

## 先看职责，不死记文件名

一个完整 Tokenizer 至少要回答：

```text
文本是否先规范化？
→ 使用什么切分模型和规则？
→ Token 怎样映射为 ID？
→ 哪些是特殊 Token？
→ 编码前后还要添加什么？
→ 怎样解码？
```

这些信息可能集中在一个文件，也可能分散在多个文件。

## 常见文件及其可能职责

| 文件 | 常见职责 | 备注 |
|---|---|---|
| `tokenizer.json` | Fast Tokenizer 的完整流水线定义 | 常可包含规范化、预切分、模型、后处理和解码配置 |
| `tokenizer_config.json` | 加载行为与 Tokenizer 配置 | 可能记录类名、最大长度、聊天模板等 |
| `special_tokens_map.json` | 特殊 Token 的名称映射 | 如 BOS、EOS、PAD、UNK |
| `added_tokens.json` | 后来加入的 Token | 并非每个仓库都有 |
| `vocab.json` | Token 到 ID 的词表 | 常见于一些 BPE 实现 |
| `merges.txt` | BPE 合并顺序 | 与对应词表共同使用 |
| `tokenizer.model` | SentencePiece 等二进制模型 | 可集中保存词表与切分相关信息 |
| `config.json` | LLM 架构配置 | 不是纯 Tokenizer 文件，但常包含 `vocab_size` 等配套信息 |
| `generation_config.json` | 生成策略默认设置 | 属于生成阶段，不决定基础文本切分算法 |

> [!warning]
> 上表是常见约定，不是强制标准。某个仓库可能缺少其中一些文件、使用别的名字，或把多项内容合并保存。判断具体模型时要看官方仓库和加载代码。

## 【训练阶段】这些文件怎样出现

```text
训练语料与人工配置
→ Tokenizer Trainer
→ 词表、规则、特殊 Token 等产物
→ 保存为一种或多种文件
```

若先训练 Tokenizer、再训练 LLM，LLM 的训练数据会始终用这些文件编码。此后每个 ID 在模型参数中逐渐获得统计意义。

## 【运行阶段】文件怎样参与请求

```text
加载模型目录
→ 构造 Tokenizer 实例
→ 编码用户与系统消息
→ 检查长度、截断或填充
→ 把 input_ids 交给模型
```

如果缺少文件或加载了错误版本，轻则特殊 Token、空格和长度行为不同，重则 ID 与 Embedding 完全错位。

## 【两阶段共同】配置一致性的三项检查

1. **词表一致**：同一 Token 对应同一 ID。
2. **词表规模匹配**：Tokenizer 产生的 ID 必须落在模型可用的 Embedding 行范围内。
3. **协议一致**：BOS、EOS、消息边界、聊天模板等必须符合模型训练时的格式。

例如 `config.json` 中的 `vocab_size: 151936` 通常说明模型参数为这个词表规模预留了相应输出或 Embedding 维度；它不直接告诉你每个 ID 对应什么文字，那要看 Tokenizer 资产。

## 常见误解

- **“只有 `vocab.json` 就能完整复现。”** 未必，还可能缺合并、规范化、后处理和特殊 Token 规则。
- **“`generation_config.json` 负责分词。”** 它主要配置生成策略，不是 Tokenizer 词表。
- **“文件名相同，内容就兼容。”** 兼容性由内容和模型契约决定。
- **“ID 没超过 `vocab_size` 就一定正确。”** 数值范围正确不代表 ID 的语义映射正确。

## 理解检查

1. `vocab.json` 与 `merges.txt` 各自解决什么问题？
2. 为什么 `config.json` 里的 `vocab_size` 不能代替真实词表？
3. 除 ID 范围外，特殊 Token 协议为什么也必须匹配？

