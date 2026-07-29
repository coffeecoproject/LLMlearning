---
type: concept
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-序列构造概览|序列构造概览]]"
next: "[[02-切段与文档边界怎样处理|切段与文档边界怎样处理]]"
tags: [llm, training, tokenizer, input-ids]
---

# 固定 Tokenizer 怎样编码训练数据

> [!summary]
> 训练数据准备阶段使用模型配套的固定 Tokenizer，把每份文本稳定地转换成 Token 和 Token ID；Tokenizer 本身通常不会跟着每个训练 Step 改变。

## 先确认当前阶段

```text
Tokenizer 构建阶段
语料 → 词表、切分规则、Special Tokens

【本篇：LLM 训练数据构造阶段】
固定 Tokenizer + 干净文本 → Token ID 序列

LLM 参数训练阶段
Token ID 序列 → Loss → 参数更新
```

本篇只讲中间这一段。

## 为什么要固定 Tokenizer

Embedding Matrix 的每一行都与某个 Token ID 对应：

```text
Token ID 125
↔ Embedding Matrix 第 125 行
```

如果训练途中随意改变词表或 ID 映射，同一个 ID 可能突然代表另一个 Token，已经学到的 Embedding 对应关系就会被破坏。

因此常见训练配方会先确定：

- Vocabulary；
- Token 到 ID 的映射；
- BOS、EOS、PAD 等 Special Token；
- 文本规范化和切分规则。

然后用这一套稳定资产编码全部训练数据。

## 一个可观察的例子

> [!example] 教学示意
> 下列 Token 和 ID 均为虚构，只说明过程。

```text
文本：我喜欢苹果。

Tokenizer 切分：
[我] [喜欢] [苹果] [。]

Token ID：
[31, 508, 9021, 7]
```

训练管线保存和传递的主要是数字序列，而不是把中文字符串直接送进 Embedding。

## 不同内容为什么会得到不同长度

Tokenizer 的单位通常是 Subword、字符片段、字节片段或特殊 Token，不等于自然语言中的“一个词”。因此：

```text
中文、英文、数字、代码、Emoji
→ 切分粒度可能不同
→ 相同字符数不代表相同 Token 数
```

这会直接影响：

- 训练语料的 Token 总量；
- 单个样本可以容纳多少原始文本；
- 某种语言占用的训练预算；
- 切段与 Packing 的结果。

## Special Token 在训练数据中的作用

训练配方可能使用：

| Special Token | 常见作用 | 是否一定存在 |
|---|---|---|
| BOS | 标记序列开始 | 否 |
| EOS | 标记序列或文档结束 | 常见但规则不同 |
| PAD | 补齐 Batch 中较短序列 | 有些模型复用 EOS 或尽量避免 Padding |
| 对话边界 Token | 区分 System、User、Assistant 等角色 | 主要用于 Chat/SFT 数据 |

这些 Token 的具体 ID 必须来自模型配套 Tokenizer 配置，不能自行猜测。

## 编码后还不能直接训练

得到 Token ID 后，通常还要：

```text
标注文档边界
→ 处理过长序列
→ 拼接短序列
→ 补齐 Batch
→ 构造 Labels 与 Mask
```

所以 Tokenize 是序列构造的入口，不是训练样本构造的终点。

## 常见误解

- **“固定 Tokenizer 就是冻结 Embedding。”** Tokenizer 不变，但 Embedding 参数可以在 LLM 训练中更新。
- **“Token ID 是文本的通用编码。”** ID 只在对应 Tokenizer 词表中有意义。
- **“同一个模型家族的所有版本一定共用 Tokenizer。”** 必须以具体模型资产为准。
- **“Tokenizer 输出后就能计算 Loss。”** 还需构造序列、Labels 与 Mask。

## 理解检查

1. 为什么训练中途随意改变 Token ID 映射会破坏已有参数关系？
2. 固定 Tokenizer 与冻结 Embedding 有什么区别？
3. 为什么同样长度的中文和英文不一定占用相同训练 Token 数？
