---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-Reward-Model与偏好评分概览|Reward Model 与偏好评分概览]]"
previous: "[[03-偏好对怎样变成Reward-Model训练样本|偏好对怎样变成 Reward Model 训练样本]]"
next: "[[05-Reward-Model训练完成后怎样被使用|Reward Model 训练完成后怎样被使用]]"
tags: [llm, reward-model, ranking-loss, gradient, optimization]
---

# Ranking Loss 怎样让 Chosen 得分更高

> [!summary] 一句话理解
> Ranking Loss 不要求人工规定每个回答必须得到几分，而是检查 Reward Model 有没有让 Chosen 排在 Rejected 前面；排错时，梯度推动参数朝正确排序方向调整。

## Ranking Loss 是什么

**Ranking（排序）**关注候选之间的先后关系；**Loss（损失）**表示当前模型输出与训练目标之间的差距。

对一条偏好对，目标很简单：

```text
Chosen 的分数 > Rejected 的分数
```

Ranking Loss 就是把“当前顺序错了多少”转换成可用于 Backward 的训练误差。

## 一个极简数字过程

以下数字只用于说明趋势，不代表真实模型的固定分数范围。

### 第一次 Forward：顺序错了

```text
score_chosen   = 0.4
score_rejected = 0.7
```

差值为：

```text
0.4 - 0.7 = -0.3
```

Chosen 反而更低，训练目标不满意，因此产生较明显的 Loss。

### Backward 与参数更新

Ranking Loss 经过 Backward 形成 Gradient，Optimizer 对 Reward Model 的 Weight 做小幅更新。

不是直接手工把数据库里的两个数字改掉，而是调整生成这两个分数的共享参数。

### 后续再次遇到相似样本

模型可能逐渐变成：

```text
score_chosen   = 1.3
score_rejected = 0.2
```

差值为：

```text
1.3 - 0.2 = 1.1
```

现在排序符合标签。训练会继续处理其他样本，避免模型只会判断这一对固定回答。

## 为什么不直接规定 Chosen=1、Rejected=0

有些分类方法确实可以使用固定类别标签，但偏好建模更关心：

```text
同一条件下，哪一个更好
```

人类通常很难稳定回答“这条文本绝对是 0.83 分”，却更容易比较 A 和 B 哪个更好。

成对比较还能表达：

```text
A 比 B 好
B 比 C 好
```

而不必先发明一个所有任务通用的绝对评分尺。

## 参数到底学到了什么

Reward Model 没有一个独立的“简洁参数”或“事实参数”。大量训练样本通过共享 Weight 共同影响它的内部表示。

如果许多偏好对都表示：

```text
遵循用户长度要求的回答 > 无视长度要求的回答
```

模型可能逐渐学会识别“长度要求是否被遵循”。

但如果数据经常把更长、更华丽的回答选为 Chosen，它也可能错误地学成：

```text
越长越好
```

Loss 只会忠实推动模型拟合标签，不会自动判断标签背后的标准是否合理。

## 为什么一个正确排序还不代表训练完成

模型可能在训练样本上排对，却在新样本上失败：

- 记住了常见模板；
- 依赖回答长度；
- 偏向特定语言或文风；
- 遇到专业问题时无法识别隐藏错误；
- 对训练时未见的模型输出分布不适应。

因此还要使用未参与训练的验证集，检查排序准确率、不同任务表现和偏差。

## 选读：为什么通常不只看“差值大于零”

如果只把正负当成硬开关，那么：

```text
0.5001 > 0.5000
```

和：

```text
2.0 > -1.0
```

都会被当作完全相同。实际 Ranking Loss 通常会平滑地考虑模型对这一排序有多确定，使梯度能够连续变化。

主线不需要记公式，只需保留：

```text
排序越不符合偏好
→ Loss 越推动参数修正
```

## 常见误解

### Loss 会直接修改 Reward Score

不会。Loss 通过 Backward 与 Optimizer 修改 Weight，未来 Forward 才会产生不同分数。

### Chosen 分数越高越好，没有上限

训练关注相对排序，但分数尺度和稳定性仍需控制。无限追求某个代理分数可能引出后续的 Reward Hacking。

### 排序准确率达到 100% 就说明模型理解真理

不成立。它可能记住训练样本或利用表面线索，仍需独立验证与分布外测试。

## 理解检查

1. `score_chosen < score_rejected` 时，训练目标为什么不满意？
2. Loss 是直接改两个分数，还是通过什么路径影响未来分数？
3. 为什么人类比较 A/B 往往比给出绝对 0.83 分更容易？
4. Reward Model 可能怎样利用错误的表面线索？

## 继续学习

- 上一篇：[[03-偏好对怎样变成Reward-Model训练样本|偏好对怎样变成 Reward Model 训练样本]]
- 下一篇：[[05-Reward-Model训练完成后怎样被使用|Reward Model 训练完成后怎样被使用]]
