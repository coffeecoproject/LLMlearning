---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[Block堆叠基础机制概览]]"
previous: "[[Hidden-State怎样逐层流动]]"
next: "[[为什么主干形状通常保持不变]]"
tags: [llm, transformer-block, depth, composition]
---

# 为什么要堆叠多个 Block

> [!summary]
> 一个 Block 只完成一轮信息交流和特征加工；堆叠多层让后面的 Block 能继续处理前面已经形成的上下文表示，逐步组合更复杂的关系。

## 一层不是一次完整回答

单个 Block 可以让各位置进行一次 Attention 信息汇集，再经过 FFN 加工。但复杂输入可能包含多步关系：

```text
指代
条件
否定
多句关联
代码依赖
任务要求
```

一轮更新不保证已经组合完所有关系。

## 后一层的优势是什么

Block 2 看到的不是最初 Token Embedding，而是 Block 1 已经加工后的结果：

```text
Block 1
→ 先形成一些上下文关系

Block 2
→ 在这些已形成的表示上再次交流和加工
```

这叫做逐层组合。它不要求人类能给每一层命名明确任务。

## 一个直白类比

可以把它暂时想成反复修订一份内部表示：

```text
初稿
→ 第一轮结合上下文修改
→ 第二轮在修改稿上继续处理
→ ……
```

> [!warning] 类比边界
> 模型没有可读的自然语言草稿，也不是每一层都完成一次人类式审稿。真实过程是数值张量经过不同参数变换。

## 层数更多一定更好吗

不一定。

增加 Block 通常会增加：

- 模型参数容量；
- 每个 Token 的计算量；
- 训练难度和资源需求；
- 运行延迟和显存压力。

最终能力还取决于宽度、数据、训练方法、Attention/FFN 结构和优化质量。不能只按层数给模型排序。

## 深度与宽度不要混淆

```text
num_hidden_layers
→ 堆叠多少个 Block，表示深度

hidden_size
→ Hidden State 主干有多宽

intermediate_size
→ 每个 Block 内 FFN 临时展开多宽
```

这三个配置描述不同方向，不能互相替代。

## 常见误解

- “每层只是重复完全相同的结果。”——输入状态和通常的参数都不同。
- “层越多一定越聪明。”——模型能力由整体设计和训练共同决定。
- “深度就是向量维度。”——Block 数量与 `hidden_size` 是不同概念。
- “每层必然对应一个清晰的语言步骤。”——功能通常分布且重叠。

## 理解检查

1. 为什么 Block 2 的输入比初始 Embedding 更有上下文？
2. `num_hidden_layers`、`hidden_size`、`intermediate_size` 分别表示什么？
3. 为什么不能只看层数判断模型能力？

下一篇：[[为什么主干形状通常保持不变|为什么主干形状通常保持不变]]。
