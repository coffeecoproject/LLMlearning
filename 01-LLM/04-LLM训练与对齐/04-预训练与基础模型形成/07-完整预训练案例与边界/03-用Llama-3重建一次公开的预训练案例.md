---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-完整预训练案例与边界概览|完整预训练案例与边界概览]]"
previous: "[[02-模型卡技术报告配置文件和权重分别能证明什么|模型卡、技术报告、配置文件和权重分别能证明什么]]"
next: "[[04-预训练能形成什么又不能保证什么|预训练能形成什么，又不能保证什么]]"
tags: [llm, pretraining, llama-3, open-model, worked-example]
---

# 用 Llama 3 重建一次公开的预训练案例

> [!summary] 一句话理解
> Llama 3 的公开资料能够验证“架构与 Tokenizer → 数据准备 → 大规模预训练 → 长上下文训练 → Base Model → 后训练模型”这条主线，但仍不足以让外部人员逐 Token 完全复现原始训练。

## 所属阶段

这是 **基于一手公开资料的案例复习**。

本篇主要参考 Llama 3 技术报告、Llama 3.1 官方模型卡和官方仓库。事实核对日期为 2026-07-29；后续版本不能自动套用这里的数字。

## 为什么选择 Llama 3.1

它适合作为教学案例，因为 Meta 同时公开了：

- 预训练与 Instruction-Tuned 版本；
- 8B、70B 和 405B 等规模；
- 官方 Model Card；
- 较完整的技术报告；
- Tokenizer、上下文和训练数据高层信息；
- 模型 Weight 和使用代码。

这让我们能把文件、论文和训练概念对应起来。

## 第一阶段：先确定模型家族

Llama 3.1 官方模型卡将其描述为自回归语言模型，提供：

```text
8B
70B
405B
```

技术报告明确说明 405B 是 Dense Transformer。Llama 3.1 家族使用 GQA，以改善推理可扩展性。

这些公开事实对应已经学过的：

```text
Decoder-only
→ Causal Self-Attention
→ 自回归下一 Token 生成

Dense
→ 每个 Token 使用共同的 FFN 路径

GQA
→ 多个 Query Head 分组共享较少的 KV Head
```

> [!source]
> 来源：[The Llama 3 Herd of Models](https://ai.meta.com/research/publications/the-llama-3-herd-of-models/)、[Llama 3.1 官方模型卡](https://github.com/meta-llama/llama-models/blob/main/models/llama3_1/MODEL_CARD.md)。核对日期：2026-07-29。

## 第二阶段：先建立 Tokenizer

Llama 3 相比 Llama 2 更换了 Tokenizer，官方说明其 Vocabulary 约为 128K，并采用基于 tiktoken 的方案。

这里的 `128K Vocabulary` 与 `128K Context` 是两个不同数字：

```text
128K Vocabulary
→ 可映射的 Token 种类数量

128K Context
→ 一次序列可容纳的位置数量
```

Tokenizer 更高效意味着同一段文本可能使用更少 Token，但它不会单独产生知识和推理能力。

## 第三阶段：准备预训练数据

Meta 官方模型卡说明 Llama 3.1 在约 15T 以上 Token 上预训练，数据来自公开可获得来源，训练数据截止时间为 2023 年 12 月。

技术报告与官方说明还描述了数据处理方向：

- 启发式过滤；
- 质量分类；
- 安全过滤；
- 去重；
- 不同来源的数据混合；
- 使用小规模训练实验帮助选择数据配方。

放回我们学习的路径：

```text
原始公开资料
→ 清理、过滤、去重
→ 数据混合
→ Tokenizer 编码
→ 训练序列与 Batch
```

官方公开的是高层来源、规模与处理方法，不是全部原始 URL 和最终每条训练样本。

## 第四阶段：初始化并进行主体预训练

模型结构确定后，各层参数初始化，再在训练序列上反复执行：

```text
Token ID
→ Forward
→ 下一 Token Logits
→ Causal Language Modeling Loss
→ Backward
→ Optimizer 更新参数
```

Meta 报告将 405B 的预训练路线概括为：

1. Initial Pre-Training；
2. Long-Context Pre-Training；
3. Annealing。

Initial Pre-Training 是形成绝大部分基础语言、知识与代码能力的主体阶段。

## 第五阶段：扩展长上下文

Llama 3 最初的 8B 和 70B 版本采用 8K 上下文。Llama 3 报告说明，在 405B 长上下文阶段中，训练长度从 8K 分六个阶段逐步增加到 128K，并使用约 800B Token。

```text
已有基础模型能力
→ 逐步扩大训练长度
→ 加入更长序列
→ 参数继续更新
→ 形成更长范围的信息利用能力
```

最终 Llama 3.1 官方模型卡为 8B、70B 和 405B 都标出 128K 上下文。

这个过程再次验证：上下文长度不是只改 `config`，而是需要训练阶段配合。

## 第六阶段：Annealing 是什么位置

Annealing 可以在这里先直白理解为预训练末期的收尾阶段：学习率逐渐降低，并使用更高质量的数据混合，帮助模型在结束训练前稳定和改善重点能力。

它仍然属于参数训练：

```text
高质量数据
→ Forward / Loss / Backward
→ 较谨慎地继续更新参数
```

它不是把模型“烧制”成固定程序，也不是 Chat 指令对齐的另一种名称。

## 第七阶段：得到 Pretrained Base Model

完成这些阶段后，形成 Llama 3.1 的 Pretrained 版本。

官方仓库把 Pretrained 和 Instruct 版本分别发布，这提供了直接证据：

```text
预训练结果
→ Base / Pretrained Model

Base Model
+ 后训练
→ Instruct Model
```

Base Model 可以用于续写和下游适配，但官方说明 Instruct 版本才针对多语言对话用途进行了优化。

## 第八阶段：后训练不是预训练的一部分

Llama 3.1 模型卡说明 Tuned 版本使用 SFT 和 RLHF 等方法对齐帮助性与安全性。

因此，用户感受到的以下行为不能全部归因于 15T Token 预训练：

- 按问题格式回答；
- 维持助手语气；
- 遵循系统指令；
- 适当拒绝请求；
- 使用工具调用格式。

它们还受到后训练和产品系统的影响。

## 把公开事实放回完整路线

| 学习环节 | Llama 3.1 公开资料能看到什么 |
|---|---|
| Tokenizer | 基于 tiktoken、约 128K Vocabulary |
| 架构 | 自回归 Transformer、GQA、8B/70B/405B |
| 数据规模 | 约 15T+ 预训练 Token |
| 数据来源 | 公开可获得来源的高层说明 |
| 数据处理 | 过滤、去重、质量分类和混合方向 |
| 训练目标 | 自回归语言建模主线 |
| 长上下文 | 8K 逐步扩展至 128K 的公开路线 |
| 训练结果 | Pretrained Model Weight |
| 后训练 | 单独的 Instruct 版本和对齐说明 |
| 评估 | Base 与 Instruct 的多项公开分数 |

## 哪些仍然不能完整知道

即使资料相对丰富，外部仍不能从已公开信息完整得到：

- 每条训练文本及其最终版本；
- 全部数据源的精确比例；
- 每个 Step 使用了哪些样本；
- 每项过滤规则的全部参数；
- 所有训练中断和恢复记录；
- 未公开失败实验；
- 完整训练代码与所有内部基础设施；
- 所有权重变化的逐步历史。

因此更准确的说法是：

> Llama 3.1 提供了足以理解和运行模型的重要公开资料，但不是原始训练过程的逐项完全复刻包。

## 为什么不能把案例直接套给其他模型

DeepSeek-V3 使用 MoE，Qwen2.5 有不同架构和数据配方，`gpt-oss` 也公开了不同的 Attention 与 MoE 设置。

它们可能共享：

```text
Tokenizer → 训练样本 → Forward → Loss → Backward → 参数更新
```

但不能因此假定它们共享：

- 完全相同的 Tokenizer；
- 相同参数形状；
- 相同数据比例；
- 相同上下文扩展路线；
- 相同后训练；
- 相同安全和工具系统。

## 为什么更不能外推到闭源 GPT

OpenAI 可能为某个产品模型公开能力、上下文和安全评估，但如果没有公开 Weight、Config 或完整架构，就不能从 Llama 3.1 或 `gpt-oss` 反推出它的层数、FFN、Attention 和训练 Token 数。

共同属于 Transformer 或自回归模型家族，只能支持高层原理类比，不能证明内部设定相同。

## 常见误解

### Llama 3.1 有 128K Vocabulary，所以能处理 128K 上下文

两者恰好都接近 128K，但含义完全不同：一个是 Token 种类数，一个是序列位置容量。

### 15T Token 就是 15T 条资料

不是。Token 是经过 Tokenizer 后的单位，不等于文档条数。

### 下载 Pretrained Weight 就能还原全部训练数据

不能。Weight 是训练结果，不是可逆的训练资料压缩包。

### Instruct 模型的所有能力都由 SFT 产生

也不对。基础语言和知识能力主要建立在预训练上，后训练进一步塑造任务行为。

## 理解检查

1. Llama 3.1 的 128K Vocabulary 和 128K Context 分别是什么？
2. 哪些公开事实说明它进行了专门的长上下文训练？
3. 为什么官方发布 Weight 和报告后，训练仍不能被完全复现？
4. 为什么不能用 `gpt-oss` 或 Llama 3.1 的架构推断闭源 GPT？

## 继续学习

- 上一篇：[[02-模型卡技术报告配置文件和权重分别能证明什么|模型卡、技术报告、配置文件和权重分别能证明什么]]
- 返回：[[00-完整预训练案例与边界概览|完整预训练案例与边界概览]]
- 下一篇：[[04-预训练能形成什么又不能保证什么|预训练能形成什么，又不能保证什么]]
