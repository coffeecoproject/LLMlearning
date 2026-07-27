---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[00-Output-Layer边界与复习概览|Output-Layer边界与复习概览]]"
previous: "[[01-Output-Layer不等于生成策略|Output-Layer不等于生成策略]]"
next: "[[03-Output-Layer理解检查|Output-Layer理解检查]]"
tags: [llm, forward-pass, output-layer, review]
---

# 从文本输入到 Logits 完整串联

> [!summary]
> 一次前向计算把离散文字逐步转换成 Token ID、连续向量、上下文相关 Hidden State，最后再回到与词表一一对应的 Logits；Output Layer 完成的是最后这次空间转换。

## 完整主线

```text
1. 用户文字
   “我喜欢猫”

2. Tokenizer Encode
   文字 → Token → Token ID

3. Embedding
   Token ID → 初始连续向量

4. Position 机制
   让 Attention 能利用顺序与距离

5. 多层 Transformer Block
   Attention：跨位置混合上下文
   FFN：分别加工各位置表示
   Residual：保存并累积主干信息
   Norm：稳定各阶段数值尺度

6. Final Hidden States
   每个位置形成最终上下文向量

7. Final Norm
   整理进入输出投影前的数值尺度

8. LM Head
   hidden_size → vocab_size

9. Logits
   每个位置得到整套词表候选分数
```

## 形状路线

忽略 Batch，假设：

```text
sequence_length = S
hidden_size = H
vocab_size = V
```

那么：

```text
Token IDs：             [S]
Embedding / Hidden：    [S,H]
Transformer 主干输出： [S,H]
Final Norm：            [S,H]
LM Head 后的 Logits：  [S,V]
```

加入 Batch：

```text
[B,S] → [B,S,H] → [B,S,V]
```

## 训练与运行从哪里分叉

```text
共同前向路线到 Logits
                 │
        ┌────────┴────────┐
        ↓                 ↓
训练：多个有效位置       运行：当前最后有效位置
→ 与目标 Token 对照      → 生成策略选择 Token
→ Loss                   → 加入上下文继续
```

## 每个主要组件改变了什么

| 组件 | 主要改变 | 通常不改变 |
|---|---|---|
| Tokenizer | 表示形式：文字到离散 ID | 模型权重 |
| Embedding | ID 变为向量 | Token 位置数 |
| Attention | 每个位置可利用的上下文信息 | 主干 `hidden_size` |
| FFN | 每个位置内部特征组合 | Token 位置间直接交流 |
| LM Head | 最后一维从 `hidden_size` 变为 `vocab_size` | Batch 和位置数量 |
| Softmax | 分数尺度变为概率分布 | 候选与词表 ID 的对应关系 |

## 一条最重要的认知

模型不是先在内部写好一句文字，再由 Output Layer 整句取出。Decoder-only LLM 通常在每一步根据当前上下文形成词表分布，选出一个 Token 后再继续下一步。整段回答是这种循环逐步形成的。

## 理解检查

1. 哪一步第一次把 Token ID 变成连续向量？
2. 哪一步把最后一维从 `hidden_size` 变成 `vocab_size`？
3. 为什么 Logits 已经与词表对应，却仍不等于显示给用户的文字？

下一篇：[[03-Output-Layer理解检查|Output Layer 理解检查]]。
