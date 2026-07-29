---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Reward-Model与偏好评分概览|Reward Model 与偏好评分概览]]"
previous: "[[01-为什么需要Reward-Model|为什么需要 Reward Model]]"
next: "[[03-偏好对怎样变成Reward-Model训练样本|偏好对怎样变成 Reward Model 训练样本]]"
tags: [llm, reward-model, input, output, reward-score]
---

# Reward Model 的输入输出是什么

> [!summary] 一句话理解
> 经典 Reward Model 通常读取“当前 Prompt 或对话上下文 + 一条完整候选回答”，再为这条回答输出一个数，表示它对该回答符合训练偏好的预测。

## 先看可观察的输入输出

以下分数只是教学示意，不是真实模型的固定尺度：

```text
输入 1：
Prompt：只用一句话解释 Embedding。
回答 A：Embedding 把离散 Token 映射为模型可计算的连续向量。

Reward Model 输出：1.4
```

```text
输入 2：
Prompt：只用一句话解释 Embedding。
回答 B：Embedding 就是把文字变成随便几个数字，越长越好。

Reward Model 输出：-0.6
```

在这个例子里，A 的分数高于 B，表示模型预测 A 更符合它从训练数据中学到的偏好。

## 输入不是只有回答文本

对话任务中，输入可能包含：

```text
System：你是面向初学者的老师。
User：只用一句话解释 Embedding。
Assistant：Embedding 把离散 Token 映射为模型可计算的连续向量。
```

为什么要带上前面的消息？因为“好不好”取决于要求：

- 用户要求一句话，长篇回答可能不合适；
- 用户要求 JSON，普通段落可能不合适；
- 用户要求中文，英文回答可能不合适；
- 上文给出了事实，回答若与上文冲突就可能较差。

所以 Reward Model 评估的是“回答是否适合这个上下文”，不是孤立地判断文字是否优美。

## 输出的一个分数怎样理解

**Scalar（标量）**就是一个数。经典偏好奖励模型常为整条候选回答产生一个 Reward Score。

需要注意：

```text
1.4 不等于 14 分
-0.6 不等于回答一定有害
0 也不必然代表及格线
```

分数的绝对值通常没有面向用户的固定含义。更可靠的理解是，在相同或相近条件下比较：

```text
Reward(A) > Reward(B)
→ Reward Model 更偏好 A
```

不同 Reward Model 的 `1.4` 通常不能直接横向比较。

## 内部怎样从 Token 得到一个数

只看直观路径：

```text
文本消息
→ Tokenizer
→ Token IDs
→ Transformer 逐层形成 Hidden State
→ 评分输出层
→ 一个 Reward Score
```

在经典实现中，Reward Model 可以使用与语言模型相似的 Transformer 骨干，但末端任务不同：

```text
语言模型 Output Layer：为下一个 Token 输出整张词表的 Logits
Reward Model 评分头：为整条回答输出一个分数
```

“评分头（Reward Head）”可以先理解为连接在模型末端、把最终内部表示转换成评分的参数层。

> [!note]
> 这是常见实现方式，不是 Reward Model 的唯一法定结构。定义上的关键是：它学习预测偏好奖励，而不是它必须采用某个固定网络形状。

## 为什么通常给整条回答评分

人类偏好标签往往比较的是两个完整回答：

```text
回答 A 整体优于回答 B
```

因此经典 Reward Model 也常输出整条回答的总体分数。它未必知道究竟是哪一个 Token 导致回答变差。

有些系统会设计过程级奖励，对中间步骤、代码测试或工具调用分别评分；那属于更细粒度的奖励设计，不是本专题的主线。

## Reward Model 和语言模型的角色不同

| 模型角色 | 主要输入 | 主要输出 | 主要用途 |
|---|---|---|---|
| 语言模型 | 上下文 Token | 下一 Token 的 Logits | 生成回答 |
| Reward Model | 上下文 + 完整候选回答 | Reward Score | 评估回答偏好 |

二者可能使用相似的 Transformer 结构，但“结构相似”不等于“承担同一个任务”。

## 常见误解

### Reward Score 就是正确率

不是。它首先是对训练偏好的预测，可能受到风格、长度和标注偏差影响。

### Reward Model 会改写低分回答

经典 Reward Model 负责评分，不负责生成或修改；生成和更新由其他模型与训练算法完成。

### 每个 Token 都有一个人工标注分数

经典偏好数据通常只说明完整回答之间谁更好，并不要求人工逐 Token 标分。

## 理解检查

1. 为什么同一句“3”在不同 Prompt 下得分可能完全不同？
2. Reward Score 的绝对值为什么不适合当作通用百分制？
3. 语言模型的 Output Layer 与 Reward Model 的评分头输出有何不同？
4. Reward Model 为什么通常需要看到完整候选回答？

## 继续学习

- 上一篇：[[01-为什么需要Reward-Model|为什么需要 Reward Model]]
- 下一篇：[[03-偏好对怎样变成Reward-Model训练样本|偏好对怎样变成 Reward Model 训练样本]]
