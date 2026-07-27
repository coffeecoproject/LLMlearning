---
type: concept
module: 1
status: complete
audience: non-specialist
parent: "[[Residual基础机制概览]]"
previous: "[[为什么不能只保留子层新结果]]"
next: "[[Residual与Normalization概览]]"
tags: [llm, transformer, residual-connection, attention, ffn]
---

# Residual 怎样连接 Attention 与 FFN

> [!summary]
> 一个常见 Transformer Block 通常有两次 Residual 更新：先叠加 Attention 产生的上下文变化，再叠加 FFN 对各位置表示产生的特征变化。

## 把一个 Block 想成两轮修改

先不考虑 Norm 的具体位置，一个常见串行 Block 可以这样读：

```text
第 1 轮：Attention 查看其他可见位置后，提出上下文修改
第 2 轮：FFN 针对每个位置当前表示，再做内部特征修改
```

因此主干经历两次更新：

```text
进入 Block 的当前版本 x
→ 加入 Attention 变化
→ 得到中间版本 h
→ 加入 FFN 变化
→ 得到离开 Block 的版本 y
```

可以把 `h` 理解为“已经完成上下文汇集、还没有完成本 Block 的 FFN 处理”的中间状态。

> [!warning] 类比边界
> Attention 和 FFN 不是两个人，也不会输出中文意见。二者都是对数值张量执行的可学习计算；“两轮修改”只帮助区分它们为什么各自需要一条 Residual 通路。

## 为什么不是整个 Block 只加一次

Attention 和 FFN 是两个职责不同的子层：

```text
Attention
→ 在 Token 位置之间选择和汇集信息

FFN
→ 对每个位置已经汇集到的表示继续做特征变换
```

两个子层都会产生新的变化，因此通常各自拥有一条 Residual 通路，而不是等整个 Block 结束后只进行一次无法区分来源的相加。

## 常见 Pre-Norm Block 的数据流

许多现代 Decoder-only 模型使用 Pre-Norm 形式。先只看结构：

```text
x
├──────────────────────────────┐
└→ Norm → Attention ───────────┤
                               ↓ 相加
                              h
├──────────────────────────────┐
└→ Norm → FFN ─────────────────┤
                               ↓ 相加
                              y
```

可以写成两步：

```text
h = x + Attention(Norm(x))
y = h + FFN(Norm(h))
```

这两行不是要求现在进行公式推导，只是在标记数据从哪里来：

1. `x` 经过 Norm 和 Attention 产生上下文变化，再加回 `x`，得到 `h`；
2. `h` 经过 Norm 和 FFN 产生特征变化，再加回 `h`，得到 `y`。

## 用小向量走完两次更新

以下数字均为教学示意。一个位置进入 Block 时：

```text
x = [1.0, 2.0, 1.0]
```

先假设 Attention 分支最终产生：

```text
Attention变化 = [0.2, 0.5, -0.1]
```

第一次 Residual：

```text
h = [1.0, 2.0, 1.0] + [0.2, 0.5, -0.1]
  = [1.2, 2.5, 0.9]
```

FFN 接收以 `h` 为主干的下一步表示。假设 FFN 分支产生：

```text
FFN变化 = [-0.1, 0.2, 0.3]
```

第二次 Residual：

```text
y = [1.2, 2.5, 0.9] + [-0.1, 0.2, 0.3]
  = [1.1, 2.7, 1.2]
```

于是这个位置在一个 Block 中依次吸收了两类变化。真实模型会同时处理整条序列的所有位置，向量维度也远大于 3。

## 用一句话走完两轮更新

继续观察：

```text
我买了苹果公司的股票
```

对“苹果”位置做教学化理解：

```text
x：进入 Block 时“苹果”位置的当前表示

Attention 变化：
利用“公司”“股票”等可见位置，形成上下文相关的变化

h：
当前表示 + Attention 变化

FFN 变化：
对 h 中已经汇集的特征继续进行非线性处理

y：
h + FFN 变化，交给下一个 Block
```

这条链路展示的是职责，而不是把语义拆成几个固定字段。模型不会在 `h` 上贴一个“苹果公司”标签，下一层接收到的仍是一组连续数值。

## 整条序列的形状怎样变化

假设：

```text
sequence_length = 3
hidden_size = 4
```

常见数据形状为：

```text
输入 x：             [3,4]
Attention分支输出：  [3,4]
第一次相加得到 h：   [3,4]
FFN分支最终输出：     [3,4]
第二次相加得到 y：   [3,4]
```

FFN 内部可以暂时升到更宽的 `intermediate_size`，但在进入 Residual 相加前必须降回 `hidden_size`。因此 Residual 通路也帮助我们理解：为什么 Attention Output Projection 和 FFN Down Projection 都要把结果带回主干宽度。

## Normalization 到底放在哪里

上面的例子采用 Pre-Norm：先 Norm，再进入 Attention 或 FFN。但 Transformer 还存在 Post-Norm 等结构：

```text
Pre-Norm 示例：  x + SubLayer(Norm(x))
Post-Norm 示例： Norm(x + SubLayer(x))
```

两者都可以包含 Residual，但 Normalization 的位置不同。因此：

```text
Residual 回答：哪条旧表示与哪条子层结果相加
Norm 顺序回答：相加前还是相加后调整数值尺度
```

具体差异会在本专题后半部分学习，现在只需避免把 Pre-Norm 的代码顺序当成所有模型的唯一结构。

## 变体为什么不能破坏接口匹配

真实模型可以改变：

- Attention 使用 MHA、GQA、MQA 或 MLA；
- FFN 使用 Dense MLP 或 MoE；
- Norm 使用 LayerNorm 或 RMSNorm；
- 子层采用串行、并行或其他组织方式。

但只要某个结果要直接加入 Residual 主干，它通常就必须回到与主干兼容的形状。变体改变内部计算，不代表可以随意把不同尺寸的张量直接相加。

## 阶段边界

> [!info] 两阶段共同
> Attention Residual 和 FFN Residual 都属于模型前向结构，LLM 训练和普通运行都会执行。

> [!info] LLM 训练阶段
> Attention、FFN 和 Norm 的参数通过训练形成；Residual 直接通路还会影响梯度传播。本节不展开参数更新。

> [!info] LLM 运行阶段
> 模型使用固定参数依次完成两次更新。KV Cache 只优化 Attention 对过去 K、V 的复用，不会替代 Residual，也不会缓存完整 Block 的 Residual 结果供任意新输入直接使用。

## 常见误解

- **“一个 Block 只有一次 Residual。”** 常见串行 Block 在 Attention 和 FFN 后各有一次，但真实架构仍可能存在变体。
- **“第二次 Residual 仍然加最初的 x。”** 常见串行结构中，FFN 的主干已经是第一次更新后的 `h`。
- **“Norm 就是 Residual 的一部分。”** 二者经常相邻，但职责和计算不同。
- **“FFN 内部升维后无法使用 Residual。”** FFN 会在相加前投影回 `hidden_size`。
- **“KV Cache 可以代替 Attention Residual。”** 一个缓存过去 K/V，一个连接当前 Block 的表示，职责完全不同。

## 理解检查

1. 为什么 Attention 和 FFN 通常各有一次 Residual 更新？
2. 在常见串行结构中，第二次 Residual 为什么以 `h` 而不是最初的 `x` 为主干？
3. FFN 内部可以升维，为什么最终仍能与 Residual 主干相加？
4. Pre-Norm 与 Post-Norm 的差别是在有没有 Residual，还是 Norm 的放置顺序？

下一步：回到 [[Residual与Normalization概览]]，进入 Normalization 为什么必要。
