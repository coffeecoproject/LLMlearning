---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Token概览|Token概览]]"
previous: "[[01-为什么模型需要Token|为什么模型需要Token]]"
next: "[[03-Token可能对应哪些文本单位|Token可能对应哪些文本单位]]"
tags: [llm, token, tokenizer]
---

# Token 到底是什么

> [!summary]
> Token 是某个具体 Tokenizer 的词表中能够作为一个整体编号的离散符号。

## 把定义拆开

- **某个具体 Tokenizer**：不同模型不一定使用同一套 Token；
- **词表中**：Token 必须属于一套有限 Vocabulary；
- **整体编号**：一次编码中，一个 Token 对应一个 Token ID；
- **离散符号**：它是类别，不是连续向量；
- **符号**：它可能显示成文字，也可能对应空白、字节片段或控制标记。

## Token 不是“单词”的英文翻译

下面这些都可能是一个 Token：

```text
猫        完整汉字
苹果      多字符片段
ing       子词片段
 hello    带前导空格的片段
。        标点
<eos>     特殊 Token
```

是否真的是一个 Token，必须由具体 Tokenizer 验证，不能仅凭肉眼判断。

## Token、ID、Embedding 和 Hidden State

| 对象 | 教学例子 | 本质 |
|---|---|---|
| Token | `"苹果"` | 词表中的离散符号 |
| Token ID | `4281` | 该符号在具体词表中的编号 |
| Token Embedding | `[0.12, -0.37, …]` | 根据 ID 查到的初始向量 |
| Hidden State | 随上下文变化的向量 | 某个序列位置当前的表示 |

```text
Token → Token ID → Token Embedding → Hidden State
```

因此“Token 被转成编码”如果指的是映射到 Token ID，可以作为口语理解；但 Token ID 不是压缩后的文字编码，更不是语义向量。

## Token type 与一次出现

句子：

```text
苹果公司不卖苹果。
```

假设两个“苹果”使用相同 Token ID，它们属于同一种 Token type，但位于两个不同位置。初始 Token Embedding 可以相同，经过上下文计算后的 Hidden State 通常不同。

```text
同一个 Token 类别
≠ 每次出现都拥有相同的上下文含义
```

## 一个简单形式化表示

把 Tokenizer 记作 `T`，文本记作 `s`：

```text
T(s) = [t₁, t₂, …, tₙ]
```

意思只是：文本 `s` 被转换成长度为 `n` 的 Token 序列。这里不需要进行公式推导。

## Token 有语义吗

Token 的表面片段可能让人看出含义，但 Tokenizer 的职责只是稳定地识别和编号它。模型对语言规律的表示主要形成在学习得到的参数与上下文相关的 Hidden State 中。

所以更准确的说法是：

> Token 是语义计算的输入单位之一，但 Token 本身不等于完整、固定的词义。

## 常见误解

- **“Token 就是单词。”** 它也可能是字、子词、空白或字节片段。
- **“Token 就是 Token ID。”** 一个是符号，一个是编号。
- **“Token 本身就是向量。”** 向量要到 Embedding Lookup 之后才出现。
- **“同一 Token 永远表示同一含义。”** 同一类别在不同上下文中的 Hidden State 可以不同。

## 理解检查

1. 为什么 Token 不一定等于一个词？
2. Token 与 Token ID 的区别是什么？
3. 同一 Token ID 出现两次，为什么后续表示仍可能不同？

下一篇：[[03-Token可能对应哪些文本单位|Token 可能对应哪些文本单位]]。
