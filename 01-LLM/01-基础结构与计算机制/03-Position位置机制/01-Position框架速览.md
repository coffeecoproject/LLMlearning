---
type: framework-note
module: 1
status: complete
audience: non-specialist
parent: "[[00-Position位置机制概览|Position位置机制概览]]"
previous: "[[00-Position位置机制概览|Position位置机制概览]]"
next: "[[02-为什么必须表示顺序|为什么必须表示顺序]]"
tags: [llm, position, framework, beginner]
---

# Position 一页看懂

> [!summary]
> Position 位置机制让模型的计算能够区分顺序和距离；它回答“这些 Token 在哪里、谁在前、相隔多远”，不是回答“这个 Token 是什么”。

## 它解决什么问题

比较：

```text
小明帮助小红
小红帮助小明
```

两句话包含相同的词，但顺序改变后关系也改变。只有 Token Embedding，模型得到的是各 Token 的初始内容表示；还需要位置机制让后续计算能够利用排列关系。

## 它位于哪里

不同模型可以在不同位置加入位置影响：

```text
Absolute Position
→ 在 Transformer 输入附近加入位置表示

RoPE
→ 在 Attention 内处理 Q、K

ALiBi
→ 在 Attention 内调整 Score
```

因此不能把所有位置机制都理解成：

```text
Token Embedding + Position Embedding
```

## 它与 Causal Mask 不同

```text
Position
→ 表达顺序、位置或距离

Causal Mask
→ 规定当前位置不能读取未来位置
```

一个负责位置信息，一个负责可见权限，二者不能互相替代。

## 训练与运行

> [!info] 两阶段共同
> 位置机制属于模型前向结构，LLM 训练和运行都要使用与模型匹配的位置规则。

训练可能塑造模型怎样利用这些位置信号；普通运行不会因为输入更长就现场重新训练位置机制。

## 框架层只需记住

```text
Embedding：Token 是什么
Position：Token 在哪里、相互距离怎样
Attention：根据内容和位置关系混合信息
Causal Mask：限制未来不可见
```

## 框架层检查

1. 为什么数组已经按顺序排列，仍需要模型可利用的位置机制？
2. Position 与 Causal Mask 有什么区别？
3. RoPE 为什么不能简单画成“给 Embedding 加一个位置向量”？

能回答这三题，就可以进入 [[01-Transformer框架速览|Transformer 一页看懂]]。需要比较具体方案时，再读 [[02-为什么必须表示顺序|为什么必须表示顺序]] 和 [[03-三类位置机制对比|三类位置机制对比]]。
