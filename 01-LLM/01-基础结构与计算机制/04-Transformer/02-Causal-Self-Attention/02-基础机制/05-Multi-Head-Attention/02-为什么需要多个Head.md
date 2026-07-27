---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Multi-Head-Attention概览|Multi-Head-Attention概览]]"
previous: "[[01-Attention-Head是什么|Attention-Head是什么]]"
next: "[[03-QKV怎样组织成多个Head|QKV怎样组织成多个Head]]"
tags: [llm, multi-head-attention, attention-weight, representation]
---

# 为什么需要多个 Head

> [!summary]
> 多个 Head 让同一位置同时拥有多套可学习的匹配标准和信息汇总结果，避免整个 Attention 子层只能依赖单一的 Q/K/V 表示与单套 Attention Weights。

## 单个 Head 的限制在哪里

先使用一句完整示意：

```text
小明 | 把 | 苹果 | 给了 | 小红 | 因为 | 她 | 喜欢 | 水果
```

现在更新“她”。在 Causal Attention 中，“她”可以读取自己和左侧所有位置，右侧的“喜欢”“水果”属于未来位置，会被 Mask。一个 Head 会为可见来源产生一组 Attention Weights：

```text
来源位置： [小明,  把, 苹果, 给了, 小红, 因为,  她]
Weight：    [0.08, 0.03, 0.18, 0.06, 0.45, 0.05, 0.15]
```

这些教学数字合计为 1。它说明单个 Head 并非只能选择一个来源，也可以混合多个位置；真正的限制是：它仍然只有一套 Q/K 匹配方式、一组经过 Softmax 的来源配比和一次对应的 Value 汇总。

如果模型只使用一个 Head，就必须让这套表示同时承担所有需要区分的关系。

## 多个 Head 增加了什么

假设同一位置有两个 Head，下面数字均为教学示意：

```text
来源位置：     [小明,  把, 苹果, 给了, 小红, 因为,  她]
Head A Weight：[0.05, 0.02, 0.08, 0.03, 0.68, 0.04, 0.10]
Head B Weight：[0.08, 0.02, 0.45, 0.20, 0.08, 0.02, 0.15]
```

两行都分别合计为 1，因为每个 Head 都有自己的 Softmax。这个例子中，Head A 从“小红”取得较大份额，Head B 从“苹果”“给了”等位置取得更多份额；这只是为了展示“不同配比”，不代表真实模型永久规定了这种人类可读分工。

两个 Head 不仅使用不同 Weight，还使用各自的 Value 子表示。因此，即使两个 Head 都关注同一个来源位置，它们从该位置传递出来的数值内容也不必相同。最后模型不是在 A 和 B 中二选一，而是保留两边产生的 Context，再交给后续投影组合。

## “多种关系”应该怎样理解

自然语言中，一个 Token 位置可能同时需要利用：

- 临近词；
- 较远的指代对象；
- 句法结构；
- 分隔符或格式信息；
- 当前任务相关的内容线索。

多个 Head 为学习不同关系提供了结构空间，但这里必须谨慎：这不表示每个 Head 必然只负责其中一项，也不表示人可以提前指定其永久职责。Head 可能分工、重叠、冗余，也可能随输入改变行为。

## 为什么不直接扩大一个 Head

把单个 Head 的向量做得更宽，确实能增加表示容量，但一个 Head 对每个接收位置通常仍只执行一次 Softmax，形成一套来源 Weight 分布。多 Head 结构则让各 Head 分别执行 Softmax，显式提供多套 Q/K 匹配和多次 Value 汇总：

```text
更宽的单 Head：一套 Softmax Weight → 一条更宽的汇总路径
两个 Head：    两套 Softmax Weight → 两条并行汇总路径
```

二者不是完全等价的结构偏好。

## 多 Head 不保证什么

增加 Head 不自动保证：

- 每个 Head 都学到清晰且不同的功能；
- Attention Weight 就能完整解释模型决策；
- Head 越多，能力就线性提升；
- 模型不会出现冗余 Head；
- 所有现代模型都采用标准 MHA。

模型设计还要在表达能力、参数组织、计算和运行资源之间取舍，因此后面才会出现 MQA、GQA、MLA 等变体。

## 因果链总结

```text
单个 Head 只有一套 Q/K/V 表示与 Weight 路径
→ 复杂上下文可能需要并行的不同匹配方式
→ 引入多个 Head
→ 各 Head 产生不同 Context
→ 保留并组合这些结果
```

## 常见误解

- **“多 Head 就是把同一次 Attention 重复计算多遍。”** 各 Head 的投影表示和结果通常不同，不是机械复制同一数字。
- **“每个 Head 都有一个人工规定的名称。”** Head 的功能由学习形成，不是固定规则表。
- **“只要 Head 足够多，模型就不会漏掉信息。”** 多 Head 仍受维度、训练、上下文和其他结构限制。
- **“多个 Head 最后只选最好的一个。”** 标准 MHA 会合并全部 Head 的结果。

## 理解检查

1. 一个 Head 的 Weight 可以分给多个来源，为什么仍需要多个 Head？
2. 一个更宽的 Head 与两个 Head 各自执行 Softmax，有什么结构差别？
3. 为什么不能承诺 Head 之间一定具有清晰分工？
4. 多个 Head 最后是竞争淘汰，还是合并结果？

下一篇：[[03-QKV怎样组织成多个Head|QKV怎样组织成多个Head]]。
