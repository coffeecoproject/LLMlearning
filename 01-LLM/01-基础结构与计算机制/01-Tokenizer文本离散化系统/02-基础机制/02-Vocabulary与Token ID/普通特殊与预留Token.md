---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[Vocabulary与Token ID概览]]"
previous: "[[Token-ID是什么]]"
next: "[[词表怎样连接模型输入与输出]]"
tags: [llm, special-token, reserved-token, vocabulary]
---

# 普通、特殊与预留 Token

> [!summary]
> 普通 Token 主要表示文本内容，特殊 Token 表示模型协议结构，预留 Token 则为词表布局或未来协议保留位置；三者都属于编号体系。

## 三种角色

| 类型 | 教学示例 | 主要作用 |
|---|---|---|
| 普通 Token | `我`、`ing`、`.` | 表示普通文本或字节片段 |
| 特殊 Token | `<bos>`、`<eos>`、角色边界 | 表示序列或消息结构 |
| 预留 Token | `<reserved_42>` | 预留编号位置或未来扩展空间 |

`<bos>`、`<eos>` 只是常用示意名称，不同模型的实际名称和协议可能不同。

## 特殊 Token 仍然是 Token

特殊 Token 通常同样具有 Token ID，也可能对应模型参数行。区别主要在用途：它表达“序列开始”“一轮结束”或“消息角色”等控制结构，而不是普通自然语言内容。

同样一串可见字符是否按特殊 Token 处理，还取决于 Tokenizer 的调用配置：

```text
"<eos>" 作为允许的特殊 Token → 可能整体映射成一个特殊 ID
"<eos>" 作为普通文本       → 可能被拆成多个普通 Token
```

## 预留不等于已经学会

把一个预留 ID 改名为 `<new_tool>`，只改变了符号与编号配置，并不会自动让模型理解“调用工具”。模型还需要看到符合协议的数据，并通过训练形成相应行为。

```text
给 Token 命名 ≠ 模型学会它的用途
```

## 常见误解

- **“特殊 Token 只是普通字符串的别名。”** 它是否整体识别通常需要专门配置。
- **“预留 Token 可以随便赋予新含义。”** 编号可用不代表模型已经学过该语义。
- **“每个模型都有相同 BOS/EOS。”** 名称、ID 和使用规则由具体模型协议决定。

## 理解检查

1. 特殊 Token 与普通 Token 的共同点是什么？
2. 为什么显示为 `<eos>` 的文本不一定被编码成 EOS ID？
3. 为什么重命名预留 Token 不会自动产生新能力？

下一篇：[[词表怎样连接模型输入与输出|词表怎样连接模型输入与输出]]。
