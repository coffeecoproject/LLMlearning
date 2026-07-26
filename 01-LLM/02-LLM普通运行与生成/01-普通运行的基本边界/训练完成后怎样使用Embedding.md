---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[普通运行的基本边界概览]]"
previous: "[[Embedding参数与语义边界概览]]"
tags: [llm, embedding, inference, parameters]
---

# 训练完成后怎样使用 Embedding

> [!summary]
> 普通运行时，模型仍然通过 Token ID 查找 Embedding，但只读取训练好的参数，不会因为一次对话自动修改它们。

## 为什么运行时仍然需要 Embedding

Tokenizer 产生的是 Token ID，而 Transformer 计算需要的是向量。因此，即使模型已经完成训练，输入仍要经过：

```text
用户文本
→ Tokenizer
→ Token ID
→ Embedding Lookup
→ Token Embedding
→ 后续 Transformer 计算
```

训练结束并不意味着 Embedding 消失，而是意味着矩阵中的数值已经学习完成，可以作为模型权重被读取。

## 运行与训练的核心差别

两种阶段都会使用 Embedding Lookup，区别在于使用之后是否根据误差更新参数：

```text
【训练阶段】
查表 → 前向计算 → 计算 Loss → 反向传播 → 更新 Embedding 等参数

【普通运行阶段】
查表 → 前向计算 → 生成输出
                    不更新参数
```

因此，“Embedding 是可学习参数”是说它能在训练中被调整，不是说每次聊天都会变化。

## 当前对话为什么仍能改变模型的回答

假设用户说：

```text
接下来把 Aurora 理解为我的项目名称。
```

模型可以在当前上下文中使用这个定义，因为后续计算会结合前面的 Token，形成与上下文相关的 [[初始表示与Hidden-State概览|Hidden State]]。

但这不等于把“Aurora 是项目名称”写进了 Embedding Matrix：

```text
Embedding 参数
→ 仍然保持不变

当前请求中的 Hidden State
→ 会随上下文改变
```

所以，**在上下文中暂时表现出适应**和**通过训练修改参数**是两件不同的事。

## 必须保留的边界

本篇只解释 Embedding 在普通运行中的作用，不展开：

- 多请求 Batch；
- 模型服务与请求调度；
- KV Cache 和增量生成；
- 设备、显存与推理优化；
- 在线训练和持续学习的实现。

这些属于后续“推理运行与交互”或“持续学习”模块，不是理解 Embedding Lookup 的前置条件。

## 常见误解

- **“运行时不训练，所以不需要 Embedding。”** Token ID 仍需先变成向量。
- **“模型接受了我的新定义，说明 Embedding 被修改了。”** 普通运行中通常只是上下文相关状态发生变化。
- **“可学习参数会在每次使用时学习。”** 参数只有进入明确的训练更新流程才会改变。
- **“推理模型在生成思考过程时会训练自己。”** 生成更多 Token 仍然可以只是固定参数下的前向计算。

## 理解检查

1. 为什么训练完成后仍然需要 Embedding Lookup？
2. 训练和普通运行都会查表，二者真正的分界是什么？
3. 模型能在当前对话使用新定义，为什么不代表 Embedding 已改变？

返回：[[普通运行的基本边界概览|普通运行的基本边界]]。
