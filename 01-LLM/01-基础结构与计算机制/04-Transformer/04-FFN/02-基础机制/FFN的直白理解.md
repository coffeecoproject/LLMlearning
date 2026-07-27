---
type: lesson
module: 1
status: complete
audience: non-specialist
parent: "[[FFN基础机制概览]]"
previous: "[[FFN基础机制概览]]"
next: "[[FFN在Transformer-Block中的位置与关联]]"
tags: [llm, ffn, beginner]
---

# FFN 的直白理解

> [!summary]
> FFN 是每个 Transformer Block 里的一台“向量加工器”：Attention 把上下文影响送到各 Token 位置后，FFN 使用训练形成的同一套规则，分别加工每个位置的向量。

## 先记一句话

```text
Attention：让各 Token 位置交换信息
FFN：让每个位置分别加工自己已经得到的信息
```

FFN 接触的不是原始文字，也不是 Token ID，而是当前 Token 位置的 Hidden State。

## 用一句话看流程

假设 Tokenizer 得到四个教学 Token：

```text
我 ｜ 买了 ｜ 苹果公司 ｜ 股票
```

经过 Attention 后，每个位置都有自己的上下文相关向量：

```text
“我”的向量       → FFN → “我”的变化向量
“买了”的向量     → FFN → “买了”的变化向量
“苹果公司”的向量 → FFN → “苹果公司”的变化向量
“股票”的向量     → FFN → “股票”的变化向量
```

同一个 Block 中，各位置使用同一套 FFN 参数；因为输入向量不同，得到的变化也不同。

> [!warning] 示例边界
> FFN 不会读取“苹果公司”这个中文标签，真实 Token 切分也可能不同。模型处理的是连续数值向量，这个例子只用于说明“每个位置分别进入同一层 FFN”。

## FFN 内部最直白的三个动作

```text
1. 展开
   把 hidden_size 宽的向量组合成更宽的中间表示

2. 调节
   使用激活函数或 Gate 改变中间数值

3. 压回
   把中间表示重新组合回 hidden_size
```

形状直觉：

```text
4 个数 → 8 个数 → 处理 → 4 个数
```

它不是把 1 个 Token 变成 8 个 Token；Token 位置数不变，只是每个位置内部的向量暂时变宽。

## FFN 输出以后发生什么

FFN 输出通常不是完整替换当前位置，而是作为一次变化加入 Residual 主干：

```text
进入 FFN 前的主干 Hidden State
+ FFN 产生的变化
= 更新后的 Hidden State
```

更新结果继续进入下一个 Transformer Block。模型最后还需要 Final Norm 和 Output Layer 才会产生词表 Logits。

## FFN 没有直接做什么

FFN 本身不会：

- 重新 Tokenize；
- 计算 Attention Q、K、V；
- 在 Token 位置之间生成 Attention Weight；
- 直接选择下一个 Token；
- 调用搜索或 Agent 工具；
- 保存 KV Cache。

## 类比的边界

“向量加工器”只是帮助理解输入和输出。真实 FFN 是线性投影、激活或门控、输出投影组成的数值函数，没有工人、工作台或可读修改意见。

## 最后只记五点

1. FFN 位于每个 Transformer Block 内部。
2. 它处理 Attention 已经更新过的 Hidden State。
3. 它对每个 Token 位置分别计算。
4. 同一层各位置共享 FFN 参数，不同 Block 通常有各自参数。
5. 它输出 Hidden State 变化，不直接输出 Token。

## 理解检查

1. FFN 接收的是文字、Token ID，还是 Hidden State？
2. `4→8→4` 改变的是 Token 数量还是向量宽度？
3. 同一层共享参数为什么不意味着所有位置输出相同？
4. FFN 输出后为什么还要经过 Residual？

下一篇：[[FFN在Transformer-Block中的位置与关联|FFN 在 Transformer Block 中的位置与关联]]。
