---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-基础模型是什么概览|基础模型是什么概览]]"
next: "[[02-Base-Model已经具备什么又缺少什么|Base Model 已经具备什么，又缺少什么]]"
tags: [llm, base-model, checkpoint, pretraining]
---

# Base Model 到底是什么

> [!summary] 一句话理解
> Base Model 是模型完成大规模基础预训练后保存并选定的参数状态，它已经学到广泛的下一 Token 规律，但通常还没有专门塑造成面向用户的助手。

## 所属阶段

**预训练完成位置。** 它连接两个阶段：

```text
前面：大量预训练数据与参数更新
→ Base Model
后面：后训练、领域适配或其他派生路线
```

## 它不是一个空模型

刚初始化的模型只有结构和起始参数；Base Model 已经经历大量训练 Token 和 Optimizer Step。

因此它的参数中已经形成：

- 语言和文本结构的统计规律；
- 许多事实、概念和实体关系的预测倾向；
- 代码、数学和不同领域模式，具体程度取决于训练数据；
- 根据上下文继续文本的能力。

Base Model 的“Base”表示后续还可以在它上面继续塑造，不表示它只有最基础或很弱的能力。

## 为什么说它是一组参数状态

训练过程中会保存多个 Checkpoint：

```text
Step 100,000 → Checkpoint A
Step 200,000 → Checkpoint B
Step 300,000 → Checkpoint C
```

训练团队会结合训练计划、验证 Loss 和能力评估，选择某个状态作为预训练阶段的输出。这个被选定并发布或继续使用的权重，可以成为 Base Model。

因此 Base Model 不一定表示“训练能进行的最后一秒”，而是一个具有明确用途的预训练权重版本。

## 一个可运行的 Base Model 通常还需要什么

只说“模型权重”还不够。实际加载通常需要配套内容：

```text
模型权重
+ 架构配置
+ 匹配的 Tokenizer
+ 必要的加载实现
→ 可以执行 Forward 的模型
```

这些文件共同让 Runtime 知道参数形状、层数、词表和计算方式。

但这里仍要区分：

- **权重文件**：保存训练后的参数数值；
- **Checkpoint**：可能还包含用于继续训练的 Optimizer 等状态；
- **模型仓库**：用于发布权重、配置、Tokenizer 和说明文件；
- **模型服务**：加载模型后对外提供 API 的运行系统。

它们互相关联，但不是同一个对象。

## Base Model 的原始训练目标

对于常见 Decoder-only Base Model，主要预训练目标仍是：

```text
根据左侧可见 Token
→ 预测下一个 Token
```

它见过文章、代码、问答和对话等多种文本，但训练程序未必一直明确告诉它：

```text
这一段是用户命令
你现在必须作为助手回答
```

所以它可以续写问题，也可能续写另一段问题、文章、角色文本或元信息。它首先拟合的是训练文本分布。

## 开放模型观察：Qwen2.5-7B

Qwen 官方模型卡把 `Qwen2.5-7B` 明确标为：

```text
Type: Causal Language Models
Training Stage: Pretraining
```

并说明它是 Base 版本，后续可以用于 SFT、RLHF 或 Continued Pretraining。这里验证的是阶段定位，不表示所有模型必须采用完全相同的配方。

来源：[Qwen2.5-7B 官方模型卡](https://huggingface.co/Qwen/Qwen2.5-7B)，核对日期：2026-07-28。

## 常见误解

### Base Model 是还没训练的模型

不是。它已经完成主要预训练；尚未完成的通常是特定后训练或适配。

### Base Model 就是某个文件

不完整。权重是核心，但实际使用还需要架构配置、Tokenizer 和加载实现。

### 名字里没有 Chat 就一定是纯 Base Model

不一定。模型命名没有全球统一标准，应查看官方模型卡和训练阶段说明。

## 理解检查

1. Base Model 与刚初始化模型最根本的区别是什么？
2. 为什么 Base Model 可以来自训练过程中的某个选定 Checkpoint？
3. 权重文件、模型仓库和模型服务有什么区别？

## 继续学习

- 返回：[[00-基础模型是什么概览|基础模型是什么概览]]
- 下一篇：[[02-Base-Model已经具备什么又缺少什么|Base Model 已经具备什么，又缺少什么]]
