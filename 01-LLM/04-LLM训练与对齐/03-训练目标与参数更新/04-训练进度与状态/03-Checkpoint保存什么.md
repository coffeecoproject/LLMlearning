---
type: concept
module: 1
status: complete
audience: non-specialist
reading: optional
parent: "[[00-训练进度与状态概览|训练进度与状态概览]]"
previous: "[[01-Batch-Step-Epoch与Token-Budget怎样区分|Batch、Step、Epoch 与 Token Budget 怎样区分]]"
next: "[[../05-完整更新案例/00-完整更新案例概览|完整更新案例概览]]"
tags: [llm, training, checkpoint, optimizer-state, recovery]
---

# Checkpoint 保存什么

> [!summary] 一句话理解
> Checkpoint（训练检查点）是训练过程某一时刻的可恢复快照；它通常不仅有模型参数，还包括继续训练所需的优化器和进度状态。

## 所属阶段

**训练阶段为主。** 运行阶段会加载其中的模型权重，但不一定需要完整训练状态。

## 为什么需要 Checkpoint

大模型训练可能持续很久，不能假设整个过程永不中断。Checkpoint 用于：

- 机器或任务中断后继续训练；
- 保存不同训练阶段的版本；
- 比较不同 Step 的效果；
- 选择合适版本用于评估或发布。

## 常见保存内容

完整训练 Checkpoint 可能包含：

| 内容 | 作用 |
|---|---|
| Model Weights | 模型当前学到的参数 |
| Optimizer State | AdamW 等优化器积累的历史统计 |
| Learning Rate Scheduler State | 当前学习率与调度进度 |
| Step、Epoch、Token Count | 训练进行到哪里 |
| Random Number Generator State | 尽量恢复随机过程 |
| Mixed Precision Scaler State | 混合精度训练的数值缩放状态 |
| Data Loader / Sampler State | 数据读到哪里、怎样采样 |
| Training Configuration | Batch、累积次数等关键配置 |

具体项目不一定保存全部内容，名称和文件组织也会不同。

## 为什么只有模型权重还不够

模型权重足以进行普通运行，也可以从这些权重重新开始一段训练。但如果想尽量接着原训练轨迹继续，缺少优化器状态等信息会产生变化。

例如 AdamW 会参考过去梯度的统计。若只加载权重而重新创建优化器，这些历史统计会从头开始。

因此要区分：

```text
能继续训练
≠
尽量原样恢复训练现场
```

## Checkpoint、模型发布包和对话记忆不是一回事

### Checkpoint

面向训练恢复，可能很大，包含训练内部状态。

### 模型发布包

面向使用和分发，通常包含模型权重、模型配置、Tokenizer 文件、生成配置等。它未必包含 Optimizer State。

### Agent 或 Chat Session

保存用户对话、任务记录或工具结果，属于应用上下文，不是模型参数的训练 Checkpoint。

这三类文件都可能叫“保存状态”，但系统层级完全不同。

## Checkpoint 与 Tokenizer 的关系

Tokenizer 通常不是在每个训练 Step 中被梯度更新的模型参数，但要正确恢复或发布模型，仍应保留与训练匹配的 Tokenizer 配置和词表。

如果换了不兼容的 Tokenizer，即使模型参数本身没有变化，Token ID 与 Embedding 行之间的对应也可能被破坏。

## 保存越频繁越好吗

不是。保存过于频繁会占用存储空间并造成额外 I/O；保存过少则可能在中断后损失大量训练进度。项目会在恢复风险、成本和评估需求之间选择间隔。

## 常见误解

### 误解一：Checkpoint 是模型学到的文本数据库

不是。核心仍是参数和训练状态，不是把训练文本逐条存进去。

### 误解二：公开模型仓库一定包含完整训练 Checkpoint

不一定。很多公开仓库只提供可运行权重和配置，不提供训练用优化器状态或数据进度。

### 误解三：加载 Checkpoint 会恢复之前的聊天记录

不会。聊天历史属于 Agent、客户端或服务端会话系统。

### 误解四：保存了 Checkpoint 就一定能完全复现训练

不一定。还需要匹配代码、依赖、硬件行为、数据版本和分布式设置等条件。

## 理解检查

1. 为什么模型权重可以支持运行，却不一定足以原样恢复训练？
2. Checkpoint 和 Hugging Face 上常见的模型发布包有什么差别？
3. 为什么 Tokenizer 虽不由反向传播更新，仍需和模型一起保存？

## 继续学习

- 上一篇：[[01-Batch-Step-Epoch与Token-Budget怎样区分|Batch、Step、Epoch 与 Token Budget 怎样区分]]
- 下一部分：[[../05-完整更新案例/00-完整更新案例概览|完整更新案例概览]]
