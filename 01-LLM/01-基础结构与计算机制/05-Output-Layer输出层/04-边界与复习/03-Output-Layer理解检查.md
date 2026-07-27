---
type: review
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer边界与复习概览|Output-Layer边界与复习概览]]"
previous: "[[02-从文本输入到Logits完整串联|从文本输入到Logits完整串联]]"
tags: [llm, output-layer, review, exercises]
---

# Output Layer 理解检查

> [!summary]
> 如果能不背术语地解释“Final Hidden State 怎样变成词表候选、训练与运行为何在 Logits 后分叉”，就已经掌握 Output Layer 主线。

## 第一组：框架检查

1. Output Layer 位于 Transformer Block 堆叠之前还是之后？
2. 它接收 Token ID、Embedding，还是 Final Hidden State？
3. LM Head 的输入宽度和输出宽度分别由什么配置决定？
4. Logits 为什么不能直接叫概率？

## 第二组：相邻概念辨析

尝试分别解释：

```text
Final Norm
LM Head
Logits
Softmax
生成策略
Tokenizer Decode
```

检查自己是否把“数值整理”“词表投影”“原始分数”“概率转换”“Token 选择”和“文字恢复”混成同一步。

## 第三组：形状预测

假设：

```text
batch_size = 2
sequence_length = 5
hidden_size = 8
vocab_size = 20
```

请回答：

1. Final Hidden States 的形状是什么？
2. Final Norm 后形状是否改变？
3. 完整 Logits 的形状是什么？
4. 单条序列某一个位置的 Logits 形状是什么？

参考答案：

```text
1. [2,5,8]
2. 通常不变，仍是 [2,5,8]
3. [2,5,20]
4. [20]
```

## 第四组：阶段判断

判断以下步骤主要属于哪里：

| 步骤 | 两阶段共同 / 训练 / 运行 |
|---|---|
| Final Hidden State 进入 LM Head | 两阶段共同 |
| 多个位置与目标 Token 计算 Loss | 训练阶段 |
| 读取当前最后有效位置的 Logits | 运行阶段 |
| 反向传播更新 LM Head 参数 | 训练阶段 |
| Top-p 后采样下一个 Token | 运行阶段 |

## 第五组：能否预测新情况

1. 若 `vocab_size` 从 50,000 增至 100,000，普通 LM Head 的输出宽度怎样变化？
2. 两条序列 Padding 后长度相同，运行时是否都应该无条件读取数组最后一格？
3. 某候选 Logit 为负，它的 Softmax 概率是否必然为零？
4. 两个模型的 `hidden_size` 相同，是否表示它们的词表和 LM Head 一定相同？
5. 开启 Weight Tying 后，模型是否就不需要 LM Head 这项输出运算？

## 主线参考结论

```text
Transformer 产生上下文相关 Final Hidden States
→ Final Norm 整理数值尺度
→ LM Head 映射到整个词表
→ 得到 Logits
→ 训练用来算 Loss，运行用来选下一个 Token
```

如果这条因果链已经能独立讲清，可以进入 [[01-LLM/02-LLM普通运行与生成/00-概览/00-LLM普通运行与生成大纲|LLM 普通运行与生成]]，继续学习 Prompt Prefill、逐 Token 解码、KV Cache 和生成策略。
