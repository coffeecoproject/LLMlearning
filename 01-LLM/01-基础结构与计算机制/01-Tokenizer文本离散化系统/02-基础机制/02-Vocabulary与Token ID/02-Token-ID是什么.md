---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Vocabulary与Token ID概览|Vocabulary与Token ID概览]]"
previous: "[[01-Vocabulary是什么|Vocabulary是什么]]"
next: "[[03-普通特殊与预留Token|普通特殊与预留Token]]"
tags: [llm, token-id, vocabulary]
---

# Token ID 是什么

> [!summary]
> Token ID 是 Token 在某套具体词表中的整数索引，它负责稳定定位，不直接表达语义大小。

## 从 Token 到 ID

假想映射：

```text
"我"   → 2
"喜欢" → 3
"苹果" → 4
```

文本的 Token 序列：

```text
["我", "喜欢", "苹果"]
```

会转换成：

```text
[2, 3, 4]
```

这一步可以称为 Token-to-ID lookup。ID 是整数，因而可以放进模型输入张量并用于查找参数行。

## ID 数字本身通常没有语义

```text
ID 90000 > ID 100
```

不能推出前者更重要、更复杂或出现更频繁。即使两个 ID 只差 1，也不能推出相应 Token 语义相近。

ID 的准确含义是：

```text
去固定编号体系中的这个位置
```

## 简单数学表示

词表大小为 `V` 且 ID 紧密连续时：

```text
0 ≤ id < V
```

例如 `V=1000`，常见合法范围是 `0～999`。这是索引范围，不是 1000 个语义等级。

## 编码和解码的两个方向

```text
编码：Token → Token ID
解码：Token ID → Token 或字节片段 → 文本
```

完整 Decode 还可能涉及字节重组、空格恢复和特殊 Token 处理，因此不只是把肉眼看到的字符串机械拼接；具体流程见 [[07-Decode与文本恢复|Decode与文本恢复]]。

## 常见误解

- **“ID 是 Token 的压缩语义编码。”** 它首先是稳定索引。
- **“编号接近意味着语义接近。”** 语义关系不由 ID 数值距离决定。
- **“Token ID 就是 Unicode 编号。”** 两者属于不同编号体系。

## 理解检查

1. Token ID 的核心作用是什么？
2. 为什么不能比较两个 ID 的大小来判断语义？
3. `V=6` 时，紧密连续编号通常有哪些合法 ID？

下一篇：[[03-普通特殊与预留Token|普通、特殊与预留 Token]]。
