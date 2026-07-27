---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Tokenizer基础机制概览]]"
previous: "[[Token可能对应哪些文本单位]]"
next: "[[Vocabulary是什么]]"
tags: [llm, token, vocabulary, token-id]
---

# Vocabulary 与 Token ID

> [!summary]
> Vocabulary 是一套有限 Token 及其编号体系；Token ID 是其中某个 Token 的整数索引。

## 三个对象先分开

```text
Vocabulary → 整套符号和编号表
Token      → 表中的一个符号
Token ID   → 该符号在这套表中的编号
```

假想词表：

| Token ID | Token |
|---:|---|
| 0 | `<bos>` |
| 1 | `<eos>` |
| 2 | `我` |
| 3 | `喜欢` |
| 4 | `苹果` |

```text
"我喜欢苹果" → ["我", "喜欢", "苹果"] → [2, 3, 4]
```

数字只用于教学，不属于真实模型。

## 子结构与学习顺序

1. [[Vocabulary是什么|Vocabulary 是什么]]：为什么词表必须有限？
2. [[Token-ID是什么|Token ID 是什么]]：编号有什么意义，又没有什么意义？
3. [[普通特殊与预留Token|普通、特殊与预留 Token]]：词表中为什么不只有文本片段？
4. [[词表怎样连接模型输入与输出|词表怎样连接模型输入与输出]]：同一编号体系怎样连接输入行和输出候选？
5. [[词表大小配置与兼容性|词表大小、配置与兼容性]]：为什么 Tokenizer 必须和模型配套？

## 简单数学总览

若词表包含 `V` 个 Token，紧密连续编号时常写成：

```text
合法 ID：0, 1, 2, …, V-1
```

例如 `V = 6`，常见合法 ID 是 `0～5`。真实工程中还要核对 Added Tokens、预留位置和模型配置，本节后面会单独说明。

## 本节边界

这里会说明词表怎样连接 Embedding 与输出候选，但不展开向量表示、逐 Token 生成和模型训练过程。
