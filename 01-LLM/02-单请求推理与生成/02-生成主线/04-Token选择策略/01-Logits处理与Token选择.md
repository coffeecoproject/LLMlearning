---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Token选择策略概览|Token选择策略概览]]"
previous: "[[00-Token选择策略概览|Token选择策略概览]]"
next: "[[02-Greedy与Sampling|Greedy与Sampling]]"
tags: [llm, generation, logits, logits-processor]
---

# Logits 处理与 Token 选择

> [!summary]
> Logits 是模型对词表候选给出的原始分数；生成控制器可以依据参数和规则修改候选关系，随后才选出真正进入上下文的一个 Token。

## 为什么不一定直接取最高 Logit

模型本身只完成了：

```text
当前上下文 → 词表中每个 Token 的原始分数
```

产品或运行库还可能需要：

- 禁止某些无效 Token；
- 在最短长度前阻止 EOS；
- 调整分布的尖锐或平缓程度；
- 只保留较可信的一部分候选；
- 抑制某些重复模式；
- 按结构化输出约束保留合法 Token。

这些都是对“怎样使用模型输出”的控制，不会在这一刻修改模型权重。

## 一个虚构例子

```text
候选：  猫    狗    汽车   <EOS>
Logit： 3.0   2.8   0.2    1.5
```

如果系统要求至少再生成两个 Token，就可以暂时屏蔽 `<EOS>`；如果使用 Top-k=2，就只保留“猫”和“狗”；最后再通过 Greedy 或 Sampling 选一个。

> [!warning] 示例边界
> 候选和分数都是教学虚构。真实词表很大，处理器的先后顺序也取决于实现。

## 模型、生成配置和运行软件怎样分工

| 层 | 负责什么 |
|---|---|
| 模型 | 根据上下文计算原始 Logits |
| Generation Config | 描述 Temperature、Top-p、最大新 Token 数等意图 |
| 生成控制器 / Runtime | 执行分数处理、选择、停止和循环 |

因此，同一个模型在不同生成配置下可以表现得更稳定或更多样，但这不代表模型内部参数发生了变化。

## 常见误解

- Softmax、Top-p 和 Sampling 不是三个 Transformer Block。
- API 参数不一定都直接传入模型层；很多由模型外的生成循环执行。
- 概率最高的 Token 不等于事实一定正确。
- 屏蔽某个 Token 是本轮候选约束，不等于把它从 Vocabulary 删除。

## 理解检查

1. 模型和生成控制器各自输出什么？
2. 为什么屏蔽 `<EOS>` 不会修改模型词表？
3. 同一模型为何能因生成配置不同而产生不同风格？

下一篇：[[02-Greedy与Sampling|Greedy 与 Sampling]]。
