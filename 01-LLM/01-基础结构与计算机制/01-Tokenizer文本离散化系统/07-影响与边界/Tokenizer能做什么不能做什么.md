---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[影响与边界概览]]"
tags: [llm, tokenizer, boundaries, misconceptions]
---

# Tokenizer 能做什么、不能做什么

> [!summary]
> Tokenizer 是文本与 Token ID 之间的转换系统，不是理解器、知识库、推理器或答案生成器。

## Tokenizer 真正负责什么

在编码方向：

```text
原始文本
→ 按配置规范化
→ 按既定规则切分
→ 处理特殊 Token
→ 查词表
→ 输出 Token ID
```

在解码方向：

```text
Token ID
→ 查回 Token 表示
→ 按解码规则拼接
→ 恢复文本或字节
```

它还可以提供 Token 数量、字符与 Token 位置对应等辅助信息，具体能力取决于实现。

## Tokenizer 不理解语义

假设 Tokenizer 将：

```text
“银行” → [某个 Token ID]
```

这个映射本身不知道“银行”是金融机构还是河岸。上下文意义要在后续模型层中，通过 Embedding、Transformer 和训练形成的参数关系处理。

因此：

```text
成功映射 ID
≠ 理解含义
```

## Tokenizer 不存储完整知识

词表中存在“爱因斯坦”，只说明这个字符串片段有一个编号，不表示词表里包含他的生平、相对论或历史背景。

```text
Vocabulary：Token ↔ ID
模型参数：从训练中形成的大量统计关系
```

两者不能混为数据库中的“词条”和“词条解释”。

## Tokenizer 不进行推理

输入：

```text
5 个苹果拿走 2 个，还剩几个？
```

Tokenizer 只把它转换为 ID 序列。减法关系、问题意图和答案生成都由后续模型计算及生成过程承担。

## Tokenizer 不决定输出内容

输出方向的完整链路中：

```text
Transformer 产生 logits
→ 生成策略选择下一个 Token ID
→ Tokenizer decode 恢复文字
```

Tokenizer 的 `decode` 只把已经选定的 ID 转回文本，它没有决定模型要选择哪个 ID。

## Tokenizer 不负责上下文取舍

Tokenizer 可以告诉上层系统一段内容占多少 Token，但不会判断：

- 哪条历史消息最重要；
- 哪个工具结果应该删除；
- 哪些信息需要摘要；
- 用户当前真正想完成什么。

这些属于模型运行系统或 Agent 的上下文管理，不属于 Tokenizer 机制。

## 一张能力边界表

| 问题 | Tokenizer 负责吗？ | 真正所属层次 |
|---|---:|---|
| 把文本变成 Token ID | 是 | Tokenizer |
| 统计输入 Token 数 | 是 | Tokenizer |
| 把已选 ID 恢复成文本 | 是 | Tokenizer decode |
| 把 ID 变成向量 | 否 | Embedding |
| 结合上下文形成表示 | 否 | Transformer |
| 判断事实真假 | 否 | 模型能力与外部验证系统 |
| 选择下一个输出 Token | 否 | 模型 logits 与生成策略 |
| 决定删除哪段历史 | 否 | Runtime / Agent 上下文策略 |

## 常见误解

- **“Token 有语义，所以 Tokenizer 理解语义。”** Token 的意义来自模型训练后的参数关系，不是编号过程本身。
- **“decode 就是模型生成答案。”** decode 只恢复已选 ID；生成选择发生在更前面。
- **“词表越丰富，模型知识越丰富。”** 词表项不是知识条目。
- **“Tokenizer 能计数，所以也会自动处理超长上下文。”** 计数与取舍是两种不同能力。

## 理解检查

1. Tokenizer 把“苹果”映射为 ID 时，是否已经理解苹果是什么？
2. 模型答案中的下一个 Token 是由 Tokenizer 选出的吗？
3. Tokenizer 能报告长度，为什么不能决定删掉哪条消息？

