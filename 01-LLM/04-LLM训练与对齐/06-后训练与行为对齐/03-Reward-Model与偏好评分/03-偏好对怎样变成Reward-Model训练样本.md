---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Reward-Model与偏好评分概览|Reward Model 与偏好评分概览]]"
previous: "[[02-Reward-Model的输入输出是什么|Reward Model 的输入输出是什么]]"
next: "[[04-Ranking-Loss怎样让Chosen得分更高|Ranking Loss 怎样让 Chosen 得分更高]]"
tags: [llm, reward-model, preference-pair, training-sample]
---

# 偏好对怎样变成 Reward Model 训练样本

> [!summary] 一句话理解
> 一条偏好对会被整理成共享同一 Prompt 的两条输入序列，Reward Model 分别给 Chosen 和 Rejected 评分，再用两者的相对顺序产生训练误差。

## 原始偏好记录

偏好数据可能记录：

```text
Prompt：请用一句话解释 Tokenizer。

Chosen：Tokenizer 是把输入文本转换成模型所用 Token IDs，并能执行反向解码的文本处理系统。

Rejected：Tokenizer 就是把每个汉字都换成固定编号。

Label：Chosen 优于 Rejected
```

这还不是模型直接计算的数字输入，需要经过与其他模型相似的文本整理和 Tokenization。

## 第一步：组成两条完整输入

系统会让两个候选共享同一个问题：

```text
序列 A = Prompt + Chosen
序列 B = Prompt + Rejected
```

这样比较的变量主要是回答本身。若 A、B 面对不同 Prompt，就无法判断分数差异来自问题还是回答。

对话模板、角色标记和特殊 Token 仍要与模型所采用的格式一致。例如，实际序列可能包含 System、User、Assistant 的边界标记；这些细节由具体模型配置决定。

## 第二步：Tokenizer 转成 Token IDs

```text
序列 A → Tokenizer → Token IDs A
序列 B → Tokenizer → Token IDs B
```

这里使用的是 **已经准备好的 Tokenizer**。这是 Reward Model 的训练过程，不是在此时重新训练 BPE 或修改 Vocabulary。

如果两条序列长度不同，训练批次可以通过 Padding 和 Attention Mask 对齐存储；Padding 只是批处理需要，不属于回答内容。

## 第三步：同一个 Reward Model 分别评分

```text
Token IDs A → 同一个 Reward Model → score_chosen
Token IDs B → 同一个 Reward Model → score_rejected
```

“同一个”很重要：两条回答必须使用同一套 Weight 和同一评分尺度，比较才有意义。

它不是训练两个模型：

```text
一个专门给 Chosen 高分
一个专门给 Rejected 低分
```

而是训练一个模型，让它能够面对任意新候选做相对判断。

## 第四步：偏好标签变成排序目标

数据希望：

```text
score_chosen > score_rejected
```

如果当前模型输出：

```text
score_chosen   = 0.2
score_rejected = 0.8
```

顺序与标签相反，训练误差会较大。

如果输出：

```text
score_chosen   = 1.1
score_rejected = 0.1
```

顺序已经正确，训练目标更满意。

具体 Loss 公式放在实现资料中即可；主线只需理解：错误排序越严重，越需要更新 Reward Model。

## 第五步：许多样本共同改变参数

一次比较只提供一个局部方向。训练会反复处理大量不同任务：

```text
问答、摘要、代码、数学、写作、安全场景、工具调用……
```

经过 Forward、Loss、Backward 和 Optimizer 更新，Reward Model 才逐渐学会在未见样本上预测偏好。

## 一个容易忽略的边界：标签只有相对关系

偏好对通常告诉模型：

```text
A 比 B 好
```

它不一定告诉模型：

```text
A 是满分答案
B 是完全错误答案
A 比 B 精确好 2.7 分
```

因此 Reward Model 最自然学到的是相对排序，而不是一个具有统一物理单位的绝对分数。

## 如果两个候选都不好怎么办

即使两个回答都有错误，数据仍可能把相对好的一条标为 Chosen：

```text
A：遗漏一个次要条件
B：核心事实完全错误
→ A 被选为 Chosen
```

Reward Model 学到的是“A 比 B 更受偏好”，不能由此推出“A 完全正确”。这也是后续必须单独做事实与任务验证的原因。

## 常见误解

### Chosen 和 Rejected 会合并成一条让模型继续生成的文本

不是。它们通常作为两条候选输入分别评分，再比较分数。

### 每一条偏好对都要训练两个 Reward Model

不是。同一个 Reward Model 共享参数并分别处理两个候选。

### Reward Model 训练会重新训练 Tokenizer

通常不会。Tokenizer 只负责把两条文本序列转换成该模型词表中的 Token IDs。

## 理解检查

1. 为什么 Chosen 和 Rejected 必须共享同一个 Prompt？
2. 为什么必须由同一个 Reward Model 给两条回答评分？
3. 偏好标签通常提供绝对分数，还是相对顺序？
4. 为什么 Chosen 仍然可能包含错误？

## 继续学习

- 上一篇：[[02-Reward-Model的输入输出是什么|Reward Model 的输入输出是什么]]
- 下一篇：[[04-Ranking-Loss怎样让Chosen得分更高|Ranking Loss 怎样让 Chosen 得分更高]]
