---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-基础模型是什么概览|基础模型是什么概览]]"
previous: "[[03-Base与Chat-Instruct-Model有什么区别|Base 与 Chat / Instruct Model 有什么区别]]"
next: "[[../04-数据组成与能力方向/00-数据组成与能力方向概览|数据组成与能力方向概览]]"
tags: [llm, base-model, post-training, continued-pretraining, model-adaptation]
---

# 为什么 Base Model 是后训练与适配的起点

> [!summary] 一句话理解
> Base Model 已经承担了最昂贵的通用能力学习，后续路线可以在这些参数上继续塑造行为、领域知识或任务能力，而不必每次从随机参数重新训练。

## 为什么不重新从头训练

从随机参数开始，模型要先重新学习：

- 语言和文本结构；
- 常见概念与关系；
- 代码和格式模式；
- Attention、FFN 等参数怎样协同处理上下文。

这需要巨量数据和计算。Base Model 已经形成这些通用能力，后续训练可以直接在其基础上调整。

```text
通用预训练能力
+ 新阶段的数据和目标
→ 面向特定用途的新参数状态
```

这就是迁移学习的核心直觉：复用已经学到的通用表示。

## 路线一：后训练成 Instruct Model

```text
Base Model
→ SFT 学习指令与回答示例
→ 偏好优化或强化学习进一步塑造行为
→ Chat / Instruct Model
```

主要目标是让已有能力更容易按照用户请求被调用，并改善格式、偏好和行为边界。

具体的 SFT、DPO、RLHF 等方法属于后面的“后训练与行为对齐”模块。

## 路线二：继续领域预训练

```text
通用 Base Model
→ 在法律、金融、医学或代码等领域文本上继续下一 Token 训练
→ 领域 Base Model
→ 再进行领域指令训练
```

Continued Pretraining（继续预训练）仍可使用语言模型目标，但数据分布转向特定领域。

它可能增强领域模式，也可能影响原有通用能力，因此不是“只添加知识、其他部分完全不动”。

## 路线三：参数高效适配

LoRA、Adapter 等方法不必更新全部基础参数，而是训练一小部分新增或选定参数。

```text
Base Model 权重
+ 小规模可训练参数
→ 面向某个任务或领域的适配结果
```

这样可以降低训练和存储成本，但效果仍受 Base Model 能力、数据质量和适配方法限制。

## 一棵简单的派生树

```text
Base Model
├── 指令与偏好后训练 → 通用 Chat / Instruct Model
├── 领域继续预训练 → 领域 Base Model → 领域助手
├── LoRA / Adapter → 轻量任务版本
└── 研究与评估 → 观察预训练能力本身
```

现实路线可以组合，并不要求四选一。

## 从模型到产品还缺什么

即使得到 Chat / Instruct 权重，用户最终使用的产品通常还包括：

```text
模型权重
+ Tokenizer 与 Chat Template
+ Runtime 和生成参数
+ System Instructions
+ 上下文管理
+ 工具、检索和安全系统
+ API 或用户界面
→ 最终模型产品
```

所以 Base Model、Chat Model、Agent 和模型产品必须分层理解：

- Base / Chat 描述模型训练阶段和权重行为；
- Runtime 负责加载和执行模型；
- Agent 负责目标、工具、记忆和反馈循环；
- 产品还可能加入权限、监控、策略与界面。

## 这对 Agent 设计有什么意义

Agent 不是用来重新完成 Base Model 的预训练。Agent 应判断：

- 哪些能力可以直接由模型完成；
- 哪些事实需要 RAG 或工具提供；
- 哪些行为需要 Workflow、验证和权限约束；
- 哪些稳定领域倾向才值得通过后训练或适配写入参数。

理解 Base Model 的位置，可以避免把 Prompt、RAG、Fine-tuning 和 Agent 当成同一种能力增强方式。

## 常见误解

### 所有业务模型都应该从头预训练

通常没有必要。多数项目会使用已有 Base 或 Instruct Model，再决定是否适配。

### Continued Pretraining 只会增加新知识，不影响旧能力

不是。参数继续变化，可能产生迁移、干扰或遗忘。

### 有了 Instruct Model 就已经有完整 Agent

不是。模型不会仅靠权重自动获得外部工具执行、持久记忆和业务闭环。

## 理解检查

1. 为什么后训练通常从 Base Model 而不是随机参数开始？
2. Continued Pretraining 与指令后训练主要目标有什么不同？
3. 为什么 Chat Model 仍不等于完整模型产品或 Agent？

## 继续学习

- 上一篇：[[03-Base与Chat-Instruct-Model有什么区别|Base 与 Chat / Instruct Model 有什么区别]]
- 返回：[[00-基础模型是什么概览|基础模型是什么概览]]
- 下一小节：[[../04-数据组成与能力方向/00-数据组成与能力方向概览|数据组成与能力方向概览]]
