---
type: subsection-index
module: 1
status: complete
audience: non-specialist
parent: "[[00-Transformer概览|Transformer概览]]"
previous: "[[00-Residual与Normalization概览|Residual与Normalization概览]]"
next: "[[00-Block堆叠与Hidden-State流动概览|Block堆叠与Hidden-State流动概览]]"
tags: [llm, ffn, mlp, swiglu, moe]
---

# FFN / MLP

> [!summary]
> FFN 位于每个 Transformer Block 内部：Attention 汇集上下文后，FFN 使用训练形成的参数，分别加工每个 Token 位置当前的 Hidden State。

## 一句话直觉

```text
Attention：让不同位置交流信息
FFN：分别加工每个位置已经得到的信息
Residual：把加工产生的变化加回主干
```

## 它在完整路线中的位置

常见的串行 Transformer Block 可以先简化为：

```text
Block 输入
→ Attention
→ Residual
→ FFN
→ Residual
→ Block 输出
```

Norm 的准确位置和不同架构变体在后面的基础机制中再解释。第一遍只需确认：FFN 在 Block 内部，通常接在 Attention 更新之后。

## 按你的学习目标选择入口

### 只想掌握整体框架

只读：[[00-FFN框架速览概览|FFN 一页看懂]]。

这一层不要求掌握公式、矩阵形状、参数量和具体模型配置。读完能说明 FFN 的位置、输入、作用和输出即可。

### 想理解它为什么能工作

进入：[[00-FFN基础机制概览|FFN 基础机制]]。

这一层解释：

- 为什么 Attention 之后还需要 FFN；
- “逐位置处理”为什么仍然利用了上下文；
- 展开、非线性、压回分别在做什么；
- 用一组很小的数字走完一次 FFN。

### 想继续看参数和真实模型

按需进入：

1. [[00-FFN参数与深入概览|FFN 参数与深入]]：形状、投影矩阵、参数量；
2. [[00-FFN扩展结构概览|FFN 扩展结构]]：SwiGLU、MoE、Router、Expert；
3. [[00-FFN真实模型观察概览|FFN 真实模型观察]]：Qwen3、DeepSeek-V3、OpenAI gpt-oss；
4. [[00-FFN边界与复习概览|FFN 边界与复习]]：训练与运行边界、知识与记忆的误区。

## 学习结构

```text
FFN
├── 01 框架速览
│   └── FFN 一页看懂
│
├── 02 基础机制
│   ├── 直白理解
│   ├── 在 Block 中的位置与关联
│   ├── 核心机制与关键名词
│   └── 小数字完整计算
│
├── 03 参数与深入
│   ├── 输入、输出与形状
│   └── 参数量与规模
│
├── 04 扩展结构
│   ├── SwiGLU 与 GeGLU
│   └── MoE、Router、Expert 与 Top-k
│
├── 05 真实模型观察
│   └── 开放模型配置核对
│
└── 06 边界与复习
    ├── 训练、运行与部署边界
    └── FFN 与模型知识的关系
```

## 名称说明

```text
FFN = Feed-Forward Network
MLP = Multi-Layer Perceptron
```

在大模型资料中，二者经常指 Transformer Block 中承担同一职责的子层。本学习库统一使用 `FFN`；`FNN` 是更宽泛的缩写，不作为这里的主名称。

## 阶段边界

> [!info] 两阶段共同
> FFN 的前向计算在 LLM 训练和普通运行时都会发生。

> [!info] LLM 训练阶段
> 前向计算之后，训练系统根据误差更新 FFN 参数。

> [!info] LLM 运行阶段
> 参数保持固定，模型只使用已经训练好的 FFN 参数加工当前 Hidden State。

开始学习：[[00-FFN框架速览概览|FFN 一页看懂]]。
