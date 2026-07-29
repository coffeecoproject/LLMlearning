---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-完整预训练案例与边界概览|完整预训练案例与边界概览]]"
next: "[[02-模型卡技术报告配置文件和权重分别能证明什么|模型卡、技术报告、配置文件和权重分别能证明什么]]"
tags: [llm, pretraining, base-model, worked-example]
---

# 从初始参数到 Base Model 完整走一遍

> [!summary] 一句话理解
> 预训练不是把资料复制进模型，而是让固定架构中的大量参数，在海量 Token 上反复经历预测、计算误差和微小更新，最终形成能够继续文本的 Base Model。

## 所属阶段

**预训练阶段的完整串联复习。**

为了看清因果链，本篇构造一个名为 `CoffeeLM` 的教学模型。名称、规模和数字全部是虚构示意，不代表真实模型。

## 开始前：先确定要训练什么

开发团队先确定目标：

```text
训练一个 Decoder-only 文本 Base Model
主要学习中文、英文和代码
采用下一 Token 预测
```

这一步会影响后续所有选择。文本模型、多模态模型、Embedding 模型和图像生成模型不会使用完全相同的结构与数据目标。

## 第一步：确定 Tokenizer

假设团队先训练并冻结一个 Tokenizer：

```text
Vocabulary Size：50,000
```

它负责：

```text
文字
→ Token 划分
→ Token ID
```

例如：

```text
“咖啡很好喝”
→ [“咖啡”, “很”, “好喝”]
→ [1832, 91, 6204]
```

Token 和 ID 是教学示意。

在一次既定 LLM 预训练中，Tokenizer 通常保持稳定，否则相同 ID 的含义变化会破坏已经学到的 Embedding 对应关系。

## 第二步：确定模型架构

教学配置：

```text
Vocabulary Size：50,000
Hidden Size：1,024
Transformer Block：24 层
Attention：Causal Self-Attention
FFN：SwiGLU
Position：RoPE
```

这些设定确定了：

- Embedding 表有多大；
- 每个 Token 的 Hidden State 多宽；
- Q、K、V 和 FFN 参数怎样组织；
- 有多少个 Block 重复处理表示；
- 使用什么位置规则；
- 总参数量大致是多少。

此时模型有完整计算结构，但还没有有用参数。

## 第三步：初始化参数

Embedding、Attention、FFN 和输出层的 Weight 按初始化规则获得较小随机值。

```text
结构已经存在
+ 参数已经有数字
→ 可以执行 Forward
```

但模型输出接近混乱概率分布：

```text
输入：“中国的首都是”
可能预测：“窗户”“跑步”“蓝色”……
```

能计算不等于已经具有语言能力。

## 第四步：把原始资料变成训练样本

原始资料可能来自网页、书籍、代码和其他获准数据。它们需要经历：

```text
来源管理
→ 清理与去重
→ 质量和安全过滤
→ Tokenizer 编码
→ 切段与文档边界处理
→ Packing / Padding
→ Batch
```

假设某个训练序列是：

```text
[今天] [天气] [很] [好]
```

对于 Causal Language Modeling，模型在不同位置学习：

```text
[今天]           → 预测 [天气]
[今天][天气]     → 预测 [很]
[今天][天气][很] → 预测 [好]
```

训练代码可以并行计算多个位置的 Loss，但 Causal Mask 不允许某个位置偷看它自己的未来答案。

## 第五步：一次 Training Step

一次 Step 的核心路径是：

```text
Batch 中的 Token ID
→ Embedding + Position
→ 多层 Transformer Block
→ Hidden State
→ Output Layer 得到 Logits
→ 与 Labels 比较得到 Loss
→ Backward 计算 Gradient
→ Optimizer 微调参数
```

假设正确 Token 是“好”：

```text
更新前：P(好) = 0.05
更新后：P(好) = 0.051
```

数字只是为了表达“一次更新通常很小”。实际训练同时改变大量参数，也不会保证某个单独概率每次都单调增加。

## 第六步：大量 Step 怎样累积

下一批样本又会产生新的误差信号：

```text
Step 1：参数略微变化
Step 2：参数再次变化
……
Step 1,000,000：形成大量统计关系
```

不同样本共同推动参数形成可复用模式：

- 字词和语法关系；
- 上下文指代；
- 常见事实关联；
- 代码结构；
- 文体和格式；
- 一些组合与推理模式。

没有一条固定规则说“第几步写入北京这条知识”。能力分散在许多参数与激活关系中。

## 第七步：训练过程中持续评估

团队不会只看训练 Loss，还会检查：

- 未参与训练的验证数据 Loss；
- 不同语言和领域表现；
- 代码与知识评测；
- 数据污染；
- 训练是否数值异常；
- 模型扩大后是否符合预期 Scaling 趋势；
- 新阶段是否损伤已有能力。

如果训练 Loss 降低、验证 Loss 却变差，可能意味着过拟合或数据分布问题，而不是模型更可靠。

## 第八步：保存 Checkpoint

训练过程中会定期保存：

```text
模型参数
+ Optimizer 状态
+ 当前训练进度
+ 必要的随机状态和配置
```

Checkpoint 的作用包括：

- 故障后继续训练；
- 比较不同训练阶段；
- 选择后续继续训练的起点；
- 保存最终 Base Model 权重。

它不是模型的聊天记录，也不是 KV Cache。

## 第九步：得到 Base Model

预定训练完成并通过基础评估后，团队得到：

```text
Tokenizer 文件
+ 模型配置
+ 训练后的模型权重
→ CoffeeLM Base
```

这个 Base Model 已经能根据上下文预测和续写 Token，但它的基础目标仍是：

```text
根据前文，什么 Token 更可能出现？
```

它不一定稳定理解“用户是在下达指令”，也不一定采用助手式回答格式。

## Base Model 运行时发生什么

参数训练完成后，普通运行只执行：

```text
用户输入
→ Tokenizer
→ Forward
→ Logits
→ Decoding 选择 Token
→ 追加 Token 再继续
```

不会自动执行：

```text
Loss → Backward → Optimizer Step
```

所以与用户对话不会自然把新知识永久写入这个 Base Model 的 Weight。

## 为什么还要后训练

如果希望 CoffeeLM 变成聊天助手，还需要进一步使用：

- 指令—回答数据；
- SFT；
- 偏好优化或强化学习；
- 安全与拒绝训练；
- 工具调用格式训练。

完整产品还可能继续加入：

```text
Runtime
+ Context Manager
+ RAG / Memory
+ Tool
+ Agent Loop
+ Evaluation 与验收
```

模型预训练完成，只代表整套系统完成了基础层。

## 哪些通常固定，哪些持续变化

| 项目 | 一次既定预训练中通常怎样 |
|---|---|
| Tokenizer 与 Vocabulary 映射 | 通常固定 |
| Block 数与 Hidden Size | 通常固定 |
| Causal Mask 规则 | 通常固定 |
| Embedding、Attention、FFN、输出层 Weight | 持续更新 |
| Batch 中的训练 Token | 不断变化 |
| Loss 与 Gradient | 每个 Step 重新产生 |
| Optimizer 状态 | 随训练更新 |

“通常”很重要：模型扩容、词表扩展或特殊继续训练可能改变结构，但那需要额外设计，不能视为普通 Step 的自然结果。

## 常见误解

### 训练就是把原文存进参数

不是。参数学习的是有助于降低预测 Loss 的分布式规律，也可能记住部分内容，但不存在一条文本对应一个固定参数槽位。

### 训练完成后模型会自动核验所有知识

不会。下一 Token 目标没有内置事实数据库对照步骤。

### Base Model 就是用户在聊天产品中看到的模型

不一定。聊天产品通常还包括后训练模型、系统指令、Runtime、工具和安全层。

### Checkpoint 就是每轮对话的 Session

不是。Checkpoint 保存训练状态；Session 保存应用对话或任务状态。

## 理解检查

1. 随机初始化模型为什么能计算，却不能可靠续写？
2. 一次 Training Step 中，哪些环节真正改变参数？
3. Base Model 与 Chat Model 之间还缺少什么？
4. Checkpoint、KV Cache 和 Session 分别保存什么层面的状态？

## 继续学习

- 返回：[[00-完整预训练案例与边界概览|完整预训练案例与边界概览]]
- 下一篇：[[02-模型卡技术报告配置文件和权重分别能证明什么|模型卡、技术报告、配置文件和权重分别能证明什么]]
