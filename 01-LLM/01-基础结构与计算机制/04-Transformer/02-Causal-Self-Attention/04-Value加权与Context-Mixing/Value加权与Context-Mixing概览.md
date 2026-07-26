---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[Causal-Self-Attention概览]]"
previous: "[[从匹配强弱到信息权重概览]]"
next: "[[Multi-Head-Attention概览]]"
tags: [llm, value, weighted-sum, context-mixing]
---

# Value 加权与 Context Mixing

> [!summary]
> Attention Weight 只说明各来源位置应占多大系数；模型还要用这些系数缩放对应的 Value，再把结果逐维相加，才能为当前接收位置形成一个新的上下文表示。

## 这一节接在什么位置

前面已经得到一条完整链路：

```text
当前接收位置的 Query
→ 与各来源位置的 Key 比较
→ Score
→ Scaling、位置影响与 Causal Mask
→ Softmax
→ Attention Weight
```

但是 Weight 只是一组系数，例如：

```text
[0.2, 0.7, 0.1]
```

它没有携带要传递的完整内容，也还不是更新后的 Token 表示。接下来必须把 Weight 与 [[Value是什么|Value]] 配对：

```text
Weight：决定每个来源占多少
Value：提供每个来源可传递的内容向量
```

## 一眼看懂完整过程

下面所有数字均为**教学示意**，不是真实模型参数或运行结果。

假设当前位置可以读取三个来源位置：

```text
来源：       [位置 0, 位置 1, 位置 2]
Weight：     [  0.2,    0.7,    0.1 ]

V₀ = [1, 0]
V₁ = [0, 2]
V₂ = [1, 1]
```

模型先分别缩放：

```text
0.2 × V₀ = [0.2, 0.0]
0.7 × V₁ = [0.0, 1.4]
0.1 × V₂ = [0.1, 0.1]
```

再逐维相加：

```text
[0.2, 0.0] + [0.0, 1.4] + [0.1, 0.1]
= [0.3, 1.5]
```

`[0.3, 1.5]` 就是这个接收位置在**当前层、当前 Head** 中得到的上下文汇总结果。

## 为什么叫 Context Mixing

`Context` 指当前位置能够读取的上下文来源，`Mixing` 指多个来源的 Value 按不同系数组合到一个新向量中。

这里的“混合”不是：

- 把原始文字粘接起来；
- 只复制最高 Weight 对应的 Token；
- 从数据库取出一段完整答案；
- 把 Attention Weight 直接当作语义内容。

真实发生的是连续向量的缩放与相加。新向量同时受多个来源影响，因此当前位置不再只保留自己的孤立信息。

## 子内容与阅读顺序

1. [[为什么Weight要作用于Value|为什么 Weight 要作用于 Value]]：分清“选择依据”和“传递内容”。
2. [[Value加权求和怎样计算|Value 加权求和怎样计算]]：用小向量走完缩放与逐维相加。
3. [[Context-Vector是什么|Context Vector 是什么]]：明确这一步产生了什么、还没有产生什么。
4. [[为什么每个位置得到不同的Context|为什么每个位置得到不同的 Context]]：从不同 Query、不同可见范围理解逐位置混合。

## 阶段标注

> [!info] 两阶段共同
> Value 加权求和是 Transformer 前向计算的一部分，LLM 训练阶段和运行阶段都会发生。本节只解释这一步的模型内部计算，不展开梯度更新、逐 Token 生成、KV Cache、Batch 或请求调度。

## 这一节的准确边界

本节结束时得到的是：

```text
单个 Head 为各接收位置产生的上下文结果
```

它还不是：

```text
多个 Head 的合并结果
不是 Attention 子层的最终输出
不是加入 Residual 后的 Hidden State
不是下一个 Token 的概率
```

这些步骤依次属于：

- [[Multi-Head-Attention概览|Multi-Head Attention]]；
- [[Residual与Normalization概览|Residual 与 Normalization]]；
- [[Output-Layer输出层概览|Output Layer]]。

## 学完后应该能够

1. 解释为什么 Weight 不能代替 Value；
2. 用两个或三个小向量完成一次加权求和；
3. 区分 Value、Attention Weight、Context Vector 与 Hidden State；
4. 解释同一个来源位置为什么会向不同接收位置贡献不同份额；
5. 说清 Value 加权与 Context Mixing 和 Multi-Head Attention 的边界。

## 最短复述

> 每个接收位置都有自己的一组 Attention Weights；这些权重分别乘到对应来源位置的 Value 上，再把缩放后的向量相加，就得到该位置在当前 Head 中的 Context Vector。

下一节：[[Multi-Head-Attention概览|Multi-Head Attention]]。
