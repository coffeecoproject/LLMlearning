---
type: reference
module: 1
status: complete
audience: non-specialist
parent: "[[00-FFN扩展结构概览|FFN扩展结构概览]]"
previous: "[[03-Router-Expert与Top-k|Router-Expert与Top-k]]"
next: "[[00-FFN真实模型观察概览|FFN真实模型观察概览]]"
tags: [llm, moe, total-parameters, active-parameters]
---

# MoE 总参数与每 Token 激活参数

> [!summary]
> MoE 总参数包含全部 Expert 和公共组件；每 Token 激活参数只统计处理当前 Token 时实际使用的参数，两者不能混为一谈。

## 教学小模型

假设：

```text
4 个 Expert
每个 Expert 100 个参数
Top-k = 1
先忽略公共组件
```

那么：

```text
总 Expert 参数 = 4 × 100 = 400
本 Token 激活 Expert 参数 = 1 × 100 = 100
```

权重存储需要全部 400 个参数，但这个 Token 只执行其中一个 Expert。

## 为什么真实比例不能直接用 k/N

真实模型还有始终参与的：

- Attention；
- Embedding；
- Norm；
- Router；
- Shared Expert；
- Final Norm 与 Output Layer；
- 其他专属模块。

因此看到 32 个 Expert、Top-4，不能直接断言完整模型只激活总参数的八分之一。

## gpt-oss 实例

OpenAI 官方模型卡说明：

```text
gpt-oss-20b
→ 20.9B 总参数
→ 3.6B 每 Token 激活参数
→ 32 个 Expert，Top-4

gpt-oss-120b
→ 116.8B 总参数
→ 5.1B 每 Token 激活参数
→ 128 个 Expert，Top-4
```

这说明总参数与激活参数可以相差很大，但实际速度还受通信、内存、Batch 和硬件利用率影响。

> [!source]
> 来源：[OpenAI gpt-oss 官方模型卡](https://deploymentsafety.openai.com/gpt-oss/evaluation)。核对日期：2026-07-27。

## 激活参数与激活函数

```text
Activation Function
→ SiLU、GELU 等非线性函数

Active Parameters
→ 当前 Token 实际使用的参数
```

只是中文都出现“激活”，含义完全不同。

## 常见误解

- 未激活 Expert 仍需被存储。
- 激活参数少不保证速度按同比例提升。
- `Top-k / Expert 数量` 不是完整模型激活参数比例。
- 激活参数不是“经过激活函数的参数”。

下一节：[[00-FFN真实模型观察概览|FFN 真实模型观察]]。
