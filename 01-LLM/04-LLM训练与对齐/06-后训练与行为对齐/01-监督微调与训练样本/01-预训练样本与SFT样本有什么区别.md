---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-监督微调与训练样本概览|监督微调与训练样本概览]]"
previous: "[[00-监督微调与训练样本概览|监督微调与训练样本概览]]"
next: "[[02-Base-Model为什么不能直接当助手|Base Model 为什么不能直接当助手]]"
tags: [llm, pretraining, sft, chat-template, loss-mask]
---

# 预训练样本与 SFT 样本有什么区别

> [!summary] 一句话理解
> 预训练样本主要提供广泛的 Token 规律；SFT 样本把角色、指令和理想回答组织成行为示范，使 Base Model 更倾向于按要求回应。

## 先看两个样本

### 预训练

```text
苹果是一种常见水果，富含水分……
```

模型通常在许多正文位置学习预测下一个 Token。

### SFT

```text
System：你是一个简洁的助手。
User：苹果属于什么类别？
Assistant：苹果是一种水果。
```

它示范“在这种上下文中，Assistant 应怎样回答”。真实角色格式由模型配套 Chat Template 决定。

## 共同机制

两者都可能继续使用 Decoder-only LLM 的下一 Token Loss：

```text
结构化内容
→ Token IDs
→ Forward 与 Loss
→ Backward
→ 参数更新
```

SFT 仍是模型参数训练，不是外挂问答数据库。

## 关键差别

| 预训练 | SFT |
|---|---|
| 大规模文本、代码等 | 精心组织的指令与回答 |
| 形成广泛基础能力 | 塑造指令和输出行为 |
| 通常得到 Base Model | 通常从 Base Model 继续训练 |
| 大量正文位置参与 Loss | 常重点训练 Assistant 位置 |

## 为什么 Prompt 可以不计算 Loss

一种常见配方是：

```text
System 与 User：作为输入读取，但不作为模仿目标
Assistant：作为输入读取，也计算生成 Loss
```

Prompt 仍然会影响回答的 Hidden State；“不计算它的 Loss”不等于模型看不到它。

这只是常见配方，不是所有 SFT 的唯一实现。

## Chat Template 为什么重要

模型训练时看到的角色边界，应与运行时构造消息的方式匹配。格式不一致时，已经训练的行为可能难以稳定触发。

## 常见误解

- SFT 也可能改变知识和技能，但主要目标通常是任务与行为适应；
- 合成 SFT 数据不是天然正确，仍要过滤和验证；
- 只训练 Assistant Loss 不代表 User 内容没有作用；
- Base Model 与 SFT Model 的差别不只是多了一段系统提示词。

## 理解检查

1. 预训练与 SFT 为什么都能使用下一 Token 目标？
2. User Token 不计算 Loss 时，是否仍会影响回答？
3. 为什么训练和运行的 Chat Template 应保持匹配？

## 继续学习

- 返回：[[00-监督微调与训练样本概览|监督微调与训练样本概览]]
- 下一篇：[[02-Base-Model为什么不能直接当助手|Base Model 为什么不能直接当助手]]
