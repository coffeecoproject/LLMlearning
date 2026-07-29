---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-监督微调与训练样本概览|监督微调与训练样本概览]]"
previous: "[[02-Base-Model为什么不能直接当助手|Base Model 为什么不能直接当助手]]"
next: "[[04-Labels与Loss-Mask怎样作用|Labels 与 Loss Mask 怎样作用]]"
tags: [llm, post-training, sft, chat-template, training-sequence]
---

# 一条对话怎样变成 SFT 训练序列

> [!summary] 一句话理解
> SFT 不是把聊天界面的气泡直接交给模型，而是先把角色消息按照固定 Chat Template 排列成一条序列，再 Tokenize，并为需要学习的回答位置准备 Labels。

## 所属阶段

**后训练阶段。** 本篇解释训练数据构造，不讨论用户运行模型时怎样保存 Session，也不讨论偏好数据中的 Chosen 与 Rejected。

## 起点：一条结构化示范

训练数据可能先以结构形式保存：

```json
{
  "messages": [
    {"role": "system", "content": "你是一个简洁的助手。"},
    {"role": "user", "content": "Token 是什么？"},
    {"role": "assistant", "content": "Token 是模型处理文本时使用的基本离散单位。"}
  ]
}
```

这条样本表达的不是单纯事实，而是：

```text
在这个 System 约束和 User 问题下
→ Assistant 应该这样回答
```

JSON 只是数据保存形式之一，不是 Transformer 直接读取的最终形态。

## 第一步：Chat Template 加入角色边界

应用配套模板可能把它整理为：

```text
<system>
你是一个简洁的助手。
<user>
Token 是什么？
<assistant>
Token 是模型处理文本时使用的基本离散单位。
<end>
```

这些标记是教学示意，不代表 OpenAI、Qwen、Llama 或其他真实模型的实际 Special Token。

Chat Template 负责明确：

- 哪段是 System；
- 哪段是 User；
- 哪段是 Assistant；
- 每轮在哪里结束；
- 是否加入回答开始或结束标记。

## 第二步：Tokenizer 变成 Token ID

模板化文本再经过配套 Tokenizer：

```text
角色标记 + 文字内容
→ Token
→ Token ID
```

教学示意：

```text
<system>      → 9001
你是          → 120
一个          → 87
简洁          → 431
……
<assistant>   → 9003
Token         → 331
是            → 52
……
```

这些 ID 是虚构的。真实 ID 必须由具体模型的 Tokenizer 文件确认。

## 第三步：形成一条 Decoder-only 序列

最终可以简化成：

```text
[System Token]
[System 内容]
[User Token]
[User 内容]
[Assistant Token]
[Assistant 回答]
[结束 Token]
```

整条序列进入同一个 Decoder-only Transformer。模型不会先把 User 交给一个独立 Encoder，再由 Decoder 回答。

在 Assistant 回答位置，Causal Mask 仍然保证每个位置只能使用此前 Token。

## 第四步：决定哪些位置作为学习目标

一种常见做法是：

```text
System 与 User
→ 提供条件
→ 不要求模型模仿这些内容

Assistant 回答
→ 提供目标行为
→ 对回答 Token 计算 Loss
```

概念上类似：

| 序列部分 | 模型是否读取 | 是否常作为 Loss 目标 |
|---|---:|---:|
| System | 是 | 否 |
| User | 是 | 否 |
| Assistant 角色起点 | 是 | 取决于配方 |
| Assistant 回答 | 是 | 是 |
| 回答结束标记 | 是 | 常见做法是计算 |

具体哪些标记计算 Loss 由训练配方决定，不是所有模型完全相同。

## 为什么 User 不计算 Loss 仍然有作用

模型生成回答 Token 时，会通过 Attention 使用前面的 System 和 User Token：

```text
User：只回答一个数字。2 + 3 等于多少？
Assistant：5
```

虽然 User Token 不一定作为模仿目标，但它决定了正确 Assistant Token 应该是什么。

```text
Prompt 是条件
Assistant 是目标
```

没有 Prompt，回答就失去对应的问题和约束。

## 多轮对话样本怎样处理

多轮样本可能是：

```text
System：你是一个助手。
User：我叫小林。
Assistant：你好，小林。
User：我叫什么？
Assistant：你叫小林。
```

训练时可以让两个 Assistant 回答位置都产生 Loss：

```text
第一轮 Assistant Token → 学习第一次回应
第二轮 Assistant Token → 学习利用前面历史回答
```

也可以根据配方只训练最后一轮。关键是必须明确哪些位置参与 Loss，不能从消息外观自动推断。

## 工具调用样本仍然是一种结构化轨迹

示意：

```text
User：上海今天多少度？
Assistant：<tool_call>weather(location="上海")</tool_call>
Tool：28°C
Assistant：上海当前约 28°C。
```

模型可以通过 SFT 学习：

- 什么时候倾向输出工具调用；
- 工具调用使用什么格式；
- 收到 Tool Result 后怎样继续回答。

但训练样本中的工具文本不表示模型训练时真的控制了天气服务。真实执行仍由 Agent Harness 完成。

## SFT 样本可能从哪里来

### 人类示范

标注人员根据规则为 Prompt 编写理想回答。

优点是可以表达细致意图，缺点是成本高、风格和质量可能不一致。

### 已有高质量资料整理

把问答、教程、代码解释等资料整理为指令格式。需要处理版权、来源和答案可靠性。

### 强模型或教师模型合成

让能力更强的模型生成 Prompt、回答或改写结果，再通过规则、模型和人工筛选。

生成速度快，但教师模型的错误和风格也可能被复制。

### 可验证任务生成

数学题可以核对答案，代码可以运行测试，结构化输出可以用程序检查格式。

这类自动验证能提高数据可靠性，但只适用于存在明确验证方法的部分任务。

## 一条样本为什么不应该只有漂亮答案

高质量 SFT 样本还要保证：

- Prompt 本身清楚且有代表性；
- 回答事实和推理尽量正确；
- 格式符合模型真实运行模板；
- 不泄露不该学习的私人信息；
- 多轮角色顺序合法；
- 工具调用与工具结果相互一致；
- 不让某一种语气或任务占据过高比例。

模型会模仿数据中的规律，也会模仿其中的错误。

## 为什么训练与运行必须使用相容模板

训练时如果使用：

```text
<user>……<assistant>……
```

运行时却改成模型从未见过的随机格式：

```text
问题者>>>……
回答者???
```

模型可能仍依靠语言能力猜出意图，但行为触发会更不稳定。

```text
训练格式
≈ 运行格式
→ 角色和边界更容易被模型识别
```

这就是为什么模型仓库通常提供配套 Chat Template。

## 开放模型观察：Qwen2.5

Qwen2.5 技术报告说明其 SFT 数据覆盖长文本生成、数学、代码、指令遵循、结构化数据、逻辑推理、跨语言和 System Instruction 等方向，并最终构建超过 100 万条 SFT 样本。

这个数字不能说明每条样本都由人类逐字编写，也不能推出所有任务比例；报告同时描述了合成、过滤、代码验证和多种自动评分方法。

> [!source]
> 来源：[Qwen2.5 Technical Report](https://arxiv.org/abs/2412.15115)，Qwen Team，2024。核对日期：2026-07-29。

## 与预训练样本的核心区别

```text
预训练：
“下面这段真实或整理后的文本怎样继续？”

SFT：
“在明确角色与任务条件下，理想 Assistant 应怎样回应？”
```

两者都能使用下一 Token Loss，但数据组织表达的学习目标不同。

## 常见误解

### 模型训练时直接读取 JSON 的 role 字段

不一定。JSON 通常先由数据管线和 Chat Template 转成模型实际 Token 序列。

### System 和 User 不计算 Loss，所以模型看不到

错误。它们作为前文条件影响 Assistant 位置的预测。

### 所有模型都使用相同的角色 Special Token

不是。模板和 Token ID 必须以具体模型资料为准。

### 工具调用 SFT 会自动给模型现实权限

不会。它主要训练输出和理解协议，权限与真实执行位于模型外。

## 理解检查

1. 结构化 Messages 为什么还不能直接作为 Transformer 输入？
2. Chat Template 与 Tokenizer 分别做了什么？
3. User Token 不作为 Loss 目标时，为什么仍会影响参数更新方向？
4. 工具调用样本训练的是模型的哪一部分行为？

## 继续学习

- 上一篇：[[02-Base-Model为什么不能直接当助手|Base Model 为什么不能直接当助手]]
- 返回：[[00-监督微调与训练样本概览|监督微调与训练样本概览]]
- 下一篇：[[04-Labels与Loss-Mask怎样作用|Labels 与 Loss Mask 怎样作用]]
