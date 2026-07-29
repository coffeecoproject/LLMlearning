---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[00-监督微调与训练样本概览|监督微调与训练样本概览]]"
previous: "[[03-一条对话怎样变成SFT训练序列|一条对话怎样变成SFT训练序列]]"
next: "[[05-SFT怎样通过许多示范改变模型行为|SFT 怎样通过许多示范改变模型行为]]"
tags: [llm, post-training, sft, labels, loss-mask, causal-mask]
---

# Labels 与 Loss Mask 怎样作用

> [!summary] 一句话理解
> Labels 指定每个预测位置的正确下一个 Token；Loss Mask 决定哪些预测错误真正计入训练目标，使模型读取完整对话作为条件，却主要模仿 Assistant 的回答。

## 所属阶段

**后训练阶段。** 这里讨论 SFT 样本怎样产生 Loss。它和 Causal Mask、Padding Mask 都可能同时存在，但三者解决的问题不同。

## 先看最小例子

教学对话：

```text
User：2 + 3 等于多少？
Assistant：5
```

经过模板和 Tokenizer 后，简化成：

```text
[User] [2+3] [Assistant] [5] [EOS]
```

模型仍然执行下一 Token 预测：

```text
看到 [User]               → 预测 [2+3]
看到 [User][2+3]          → 预测 [Assistant]
看到 ……[Assistant]        → 预测 [5]
看到 ……[Assistant][5]     → 预测 [EOS]
```

SFT 可以只关心后两项：模型是否正确生成回答 `5`，并在正确位置结束。

## Labels 是什么

Label 可以理解为某个预测位置的“标准下一个 Token”。

```text
当前前缀：……[Assistant]
正确 Label：[5]
```

模型输出所有 Vocabulary Token 的 Logits，再通过 Loss 比较模型给 `[5]` 的概率是否足够高。

Labels 不是人工给每个参数写答案，而是给 Loss 提供正确目标。

## Loss Mask 是什么

Loss Mask 决定某个位置的预测是否计入总 Loss。

教学表格：

| 当前前缀末尾 | 应预测的下一个 Token | 是否计入 SFT Loss |
|---|---|---:|
| `[User]` | `[2+3]` | 否 |
| `[2+3]` | `[Assistant]` | 取决于配方 |
| `[Assistant]` | `[5]` | 是 |
| `[5]` | `[EOS]` | 常见做法是 |

这样训练关注的是：

```text
给定 User 条件
→ Assistant 应该生成什么
```

而不是要求模型学习“怎样生成用户的问题”。

## 常见实现中的 `-100`

很多训练框架会建立一个与输入相近的 Labels 数组，并把不需要计算 Loss 的位置设为特殊忽略值，例如 `-100`。

教学示意：

```text
Input IDs： [9002, 321, 9003, 53, 9004]
Labels：    [-100, -100, -100, 53, 9004]
```

含义是：

```text
-100 → 这个目标位置不计入 Loss
53   → 这里应学习回答 Token
9004 → 这里应学习结束 Token
```

真实框架可能在内部对 Input 与 Label 做一位 Shift，因此不要把教学数组当成所有库的精确索引规则。核心关系仍是“用前面的 Token 预测后一个 Token”。

## Loss Mask 不会让模型看不见 Prompt

这是最关键的区别。

```text
System 与 User 不计入直接 Loss
≠ 从模型输入中删除
≠ Attention 看不到
```

预测 `[5]` 时，模型仍需要读取：

```text
User：2 + 3 等于多少？
```

如果 Prompt 的表示影响了回答预测，那么回答位置的 Loss 仍会通过 Backward 影响前面计算路径所使用的参数。

所以更准确地说：Prompt 没有被要求“作为目标被模仿”，但它仍然是产生正确回答的必要条件。

## 三种 Mask 不要混淆

| 名称 | 控制什么 | 主要回答 |
|---|---|---|
| Causal Mask | 每个位置能看哪些位置 | 能否偷看未来？ |
| Padding / Attention Mask | 哪些位置是真实内容或 Padding | Padding 是否参与计算？ |
| Loss Mask | 哪些预测错误计入 Loss | 模型要模仿哪些位置？ |

### Causal Mask

即使整条答案已经放入训练张量，预测 `5` 的位置也不能偷看未来的 `5`。

### Padding Mask

不同长度样本组成 Batch 时，Padding 位置通常不应被当成真实内容。

### Loss Mask

System 和 User 可以被模型读取，但可以不作为预测训练目标。

三者可能在同一次 SFT Forward 中共同工作。

## 多轮对话怎样设置 Loss Mask

样本：

```text
User 1：我叫小林。
Assistant 1：你好，小林。
User 2：我叫什么？
Assistant 2：你叫小林。
```

一种配方：

```text
User 1       → 不计算 Loss
Assistant 1  → 计算 Loss
User 2       → 不计算 Loss
Assistant 2  → 计算 Loss
```

另一种配方可能只对 `Assistant 2` 计算 Loss。两者都可以形成合法训练数据，但学习信号数量和重点不同。

## Tool Result 是否计算 Loss

工具轨迹：

```text
Assistant：调用天气工具
Tool：上海 28°C
Assistant：上海当前约 28°C。
```

常见设计可能是：

- Assistant Tool Call：计算 Loss，让模型学习调用格式；
- Tool Result：不计算 Loss，因为这是外部环境返回；
- 最终 Assistant 回答：计算 Loss。

具体配方取决于模型和数据目标。不能看到 `Tool` 角色就假定一定采用某种 Mask。

## 为什么回答结束标记也重要

如果模型只学回答内容，却没有稳定学会在哪里结束，运行时可能出现：

- 回答后继续模拟下一条 User；
- 不必要地续写更多段落；
- 跨过本轮角色边界。

因此 EOS 或回合结束标记常被包含在 Assistant 学习目标中，让模型学会“回答到这里结束”。

## 一个可选的小计算

假设正确 Token 是 `5`。

训练前：

```text
P(5) = 0.20
Loss ≈ -ln(0.20) ≈ 1.61
```

训练后：

```text
P(5) = 0.60
Loss ≈ -ln(0.60) ≈ 0.51
```

正确 Token 概率提高，Loss 下降。实际 SFT 会对许多回答位置和多个样本求平均，不会只更新一个答案。

> [!note] 数学边界
> 这里只展示 Cross-Entropy 的直觉，不要求推导对数或 Gradient。理解“正确回答 Token 概率越高，Loss 越低”即可。

## 是否必须只训练 Assistant Token

不是绝对规则。

一些配方可能：

- 对整条序列计算 Loss；
- 对所有 Assistant 回合计算 Loss；
- 只训练最后一个 Assistant 回合；
- 对不同角色设置不同权重；
- 对推理过程与最终答案采用不同处理。

选择会影响模型在模仿什么，也需要通过评估验证。

## 常见误解

### Loss Mask 和 Causal Mask 是一回事

不是。一个决定哪些错误计分，一个决定哪些位置可见。

### Label 是回答整段文字的一个编号

不是。自回归训练通常在多个 Token 位置分别拥有下一个 Token 目标。

### Prompt 设为 `-100` 后完全不影响训练

错误。Prompt 仍是回答预测的条件，回答 Loss 会通过相关计算路径反向传播。

### 不训练 User Token，模型就不理解用户

不对。模型需要利用 User 内容才能把 Assistant Token 预测正确。

## 理解检查

1. Label 与 Loss Mask 分别负责什么？
2. 为什么 User Token 不计入 Loss，模型仍能学习响应 User？
3. Causal Mask、Padding Mask 和 Loss Mask 有什么区别？
4. Tool Result 为什么可能被读取但不作为模仿目标？

## 继续学习

- 上一篇：[[03-一条对话怎样变成SFT训练序列|一条对话怎样变成 SFT 训练序列]]
- 返回：[[00-监督微调与训练样本概览|监督微调与训练样本概览]]
- 下一篇：[[05-SFT怎样通过许多示范改变模型行为|SFT 怎样通过许多示范改变模型行为]]
